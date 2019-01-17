---
title: PowerShell MSSQL cron job
date: 2019-01-16 17:25:00 -05:00
tags:
 - SQL
 - PowerShell
 - Linux
excerpt: Setting up a cron job that runs a PowerShell script to build an htpasswd file from the results of an Mssql query
classes: wide
toc: true
toc_label: "Skip ahead"
---
# I did **WHAT**

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">
Just setup a cron job that runs a PowerShell script to build an htpasswd file from the results of an Mssql query. Cross platform ftw. Thanks <a href="https://twitter.com/cl?ref_src=twsrc%5Etfw">@cl</a>
<a href="https://twitter.com/PowerShell_Team?ref_src=twsrc%5Etfw">@PowerShell_Team</a>
<a href="https://twitter.com/hashtag/PowerShell?src=hash&amp;ref_src=twsrc%5Etfw">#PowerShell</a> <a href="https://twitter.com/hashtag/dbatools?src=hash&amp;ref_src=twsrc%5Etfw">#dbatools</a></p>&mdash;
Damien Solodow (@DSolodow) <a href="https://twitter.com/DSolodow/status/1085308282993037318?ref_src=twsrc%5Etfw">January 15, 2019</a>
</blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## **Why** I did that

TL;DR - wanted to get rid of a manual process on a legacy system. :smiley:

### Longer version

One of our few remaining legacy servers at **$WORK** is a Server 2008 R2 VM that is eating far more disk than is called for.

Just about the *only* thing the server does is host a pretty basic website. The sticky wicket though is that a few directories on the site need to require authentication. Since not all of the users have accounts in our Active Directory, the credentials are based on data from their ERP record.

The solution used on this legacy server was a 3rd party HTML "locker" that was last updated to support **Windows 8**. :open_mouth:

The process to update the credential list involves a SQL Agent job that emails a file to someone, who then has to follow a nearly 30 step **manual process** that involves carefully modifying and saving the file, logging into the web server via RDP, and running a GUI app to import said file and update the credential list. :sob: :scream: :dizzy_face:

## **How** I did that

### New solution

This sounded like a good place for a Linux VM running Apache, and using htaccess and htpasswd. I have some prior experience with Apache, and it looked like the easier path compared to using IIS and having to deal with scripting the creation/management of IIS users, or getting IIS to authenticate directly against a database.

Main reasons for that were:

1. I'm not a web developer by **any** stretch of the imagination
2. The IIS PowerShell module (WebAdministration) is *in my experience* an ugly PITA

I went with an Ubuntu LTS install, since I have prior experience there, they've been playing very nice with Microsoft of late, and it's what I have in my [WSL][wsl-link] install.

### Basic setup

So it was time to:

1. Copy web content to new server
2. Add SSL certificate to server
3. Configure Apache for site, SSL redirection
4. Create sample htpasswd for site (outside site content)
5. Configure Apache to do forms authentication for the required directories using the sample htpasswd
   1. this was in the main apache config instead of using .htaccess as it's better that way if you have control of the server
6. Test it out

*Most* of this was from apache docs and some Googling, and some trial and error.

### Getting the data

I created a SQL login in the database for the script to use, and granted it SELECT access to the appropriate view and table.
Then I used that login in [SSMS][ssms-link] to run a test query:

```sql
SELECT top 10 username, password FROM erp_db..view_user
```

This ran successfully, so credentials, security, and query are good, so time to run that via PowerShell on the Linux server.
I'd selected to install powershell-core during the initial Ubuntu install, but instructions are here: [Install PowerShell on Ubuntu][pscore-linux-link]

The SQLServer PowerShell module is cross-platform, so it can be installed in PowerShell Core on Linux. :thumbs_up:

Unfortunately, installing this revealed that only a subset of cmdlets are available on Core under Linux, and **Invoke-Sqlcmd** isn't one of them. :anguished:

I remembered a recent tweet from @cl about the dbatools module going cross-platform, so I gave that one a try:

```powershell
Install-Module dbatools -Scope CurrentUser
Get-Command -Module dbatools -Noun *query*
```

**Invoke-DbaQuery** looks like a winner, so time to read the [documentation][dbaquery-help] for it.

Looked like the commands to run were:

```powershell
$login = get-credential
Invoke-DbaQuery -SqlCredential $login -SqlInstance 'SQLSERVER' -Database 'erp_db' -Query 'SELECT top 10 username, password FROM view_user'
```

Unfortunately, I got a warning instead of results:

WARNING: [12:23:25][Invoke-DbaQuery] Failure | Property LoginSecure cannot be changed or read after a connection string has been set.

Doh! :astonished:

Maybe this was something wonky with Linux. So I tried the same PowerShell on my Windows box, using Windows PowerShell instead of Core.

Nope, same error. Smells like a bug in the module, so off to their [issues page][issues-link].

Searching through there didn't show any previous reports of this issue, so I used their handy Issues template and submitted a new one: [Issue-4946][4946-link]

**Within a couple hours** it had been confirmed, and a fix merged. :heart_eyes:

So I updated my systems to the fixed module version (0.9.741), and re-tried my query. This time I got results back! :+1::+1:

### Building htaccess from the data

Now that I could pull from the database, it was time to turn that into an htpassword file.

I gave the user that would be running the script via cron the ability to edit the htpassword file:

```bash
sudo setfacl -m u:cron_user:rw htpassword
```

This seemed a more minimal course than giving cron_user the ability to run 'sudo htpasswd' without a password.

The htpasswd utility allows you to pass both the username and password as part of the command line, so it was a simple PowerShell bit to do that:

```powershell
$login = get-credential
$creds = Invoke-DbaQuery -SqlCredential $login -SqlInstance 'SQLSERVER' -Database 'erp_db' -Query 'SELECT top 10 username, password FROM view_user'
$creds | ForEach-Object {htpasswd -b /path/to/htpassword $_.username $_.password}
```

That looked to have worked; it ran without error and I was able to use one of the username, password combos from the SQL query to login to the protected directories. :raised_hands:

Only catch was, the new lines were being *added* to the htpassword file, so I was going to need to either blank out the file or recycle it before building it; otherwise it would get oversized, and have multiple (potentially different) entries for each person. :worried:

### Fleshing it out and polishing it

Now that I had a **working** proof of concept, just had to do a few more things:

1. Flush out the file before re-filling it (Linux has a handy **truncate** command)
2. Make sure I didn't flush out the file if I didn't have data to put in
3. Add some error reporting
4. Make it pretty

So here's what it ended up like:

```powershell
#Requires -Modules @{ModuleName="dbatools"; ModuleVersion="0.9.741"}

$username = "SQLLOGIN"
$password = ConvertTo-SecureString -String "SQLPASSWORD" -AsPlainText -Force
$cred = New-Object -typename System.Management.Automation.PSCredential -ArgumentList $username, $password

$invokeDbaQuerySplat = @{
    SqlInstance   = 'SQLSERVER'
    Database      = 'erp_db'
    SqlCredential = $cred
    Query         = "SELECT username,password FROM view_user"
}
$results = Invoke-DbaQuery @invokeDbaQuerySplat
if ($results) {
    #clear file to avoid duplicate entries
    truncate --size 0 /path/to/htpassword
    $results | ForEach-Object {htpasswd -b /path/to/htpassword $_.username $_.password}
} else {
    $servername = [system.net.dns]::GetHostEntry((hostname)).hostname
    $sendMailMessageSplat = @{
        Subject    = "Script error on $servername"
        From       = 'cron@SERVER'
        To         = 'sysadmins@WORK.COM'
        SmtpServer = 'SMTPSERVER'
        Body       = "Script: $PSCommandPath  `n Threw: $Error[0]"
    }
    Send-MailMessage @sendMailMessageSplat
}
```

Success! :relieved: :sunglasses:

[wsl-link]:https://docs.microsoft.com/en-us/windows/wsl/faq
[ssms-link]:https://docs.microsoft.com/en-us/sql/ssms/sql-server-management-studio-ssms
[pscore-linux-link]:https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-core-on-linux?view=powershell-6#ubuntu-1804
[dbaquery-help]:https://docs.dbatools.io/#Invoke-DbaQuery
[issues-link]:https://github.com/sqlcollaborative/dbatools/issues
[4946-link]:https://github.com/sqlcollaborative/dbatools/issues/4946