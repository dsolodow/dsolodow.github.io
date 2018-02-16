---
title: Fun with certificates – Outlook Web Access certificate error
date: '2014-09-19T14:31:00.001-04:00'
tags:
 - Security
 - Certificates
categories: Troubleshooting
blogger_id: tag:blogger.com,1999:blog-23185081170317196.post-8303294717979752005
blogger_orig_url: http://theevolvingadmin.blogspot.com/2014/09/fun-with-certificates-outlook-web.html
excerpt: Running down a cause of odd browser SSL/HTTPS errors
toc: true
---

## The Problem

We recently had a user (we’ll call them "Bob") send in a ticket because when he tried to access Outlook Web Access from home he got a certificate error in the web browser that said the site wasn’t secure. Understandably concerned by this, he closed out the browser and contacted us.

## The Diagnosis

So the first order of business was to determine if it was a global issue or just poor Bob. Since no other tickets had come in, and the site loaded correctly for us here it was almost certainly just Bob having this issue.

### A few items that made this even more interesting

* Bob got this security warning on any secure Harrison websites, not just Outlook Web Access
* Bob didn't have problems with non-Harrison secure websites
* This included secure websites that use the same certificate provider that we do ([DigiCert](https://www.digicert.com/))
* Internet Explorer and Chrome both gave a certificate warning for Harrison sites, but Firefox worked fine.

This last point is rather interesting and is actually rather telling. IE and Chrome both use the Windows certificate store, whereas Firefox maintains its own certificate store. On a side-note, this is why Firefox is problematic with non-public Certificate Authorities; you can’t deploy the root certificates to it via GPO.

So we took a look at Bobs PC, and visited our Outlook Web Access page to get the certificate warning. We pulled up the certificate info ([HowTo](https://www.globalsign.com/en/blog/how-to-view-ssl-certificate-details/#ie)) so we could try to find out why his browser thought something was wrong.  Here’s what we saw:

![Image of certificate details showing error]({{site.url}}/assets/images/2014-09-19-fun-with-certs/cert.png)

The Details tab wasn't immediately helpful, but the Certification Path tab was. The Certification Path tab shows the [Certificate Chain](http://en.wikipedia.org/wiki/Chain_of_trust) for the given certificate:

![Image of certificate path]({{site.url}}/assets/images/2014-09-19-fun-with-certs/certpath.png)

Our certificate (*.harrison.edu) said that it was OK. The next one up the chain (DigiCert Secure Server CA) also said it was OK. However, the top (or root) certificate was not OK.

Ah-ha! This meant that the Windows certificate store had a corrupt copy of that certificate that needed to be replaced.

## The Solution

1. Determine which certificate in the store needs to be replaced
2. Get a good copy of the certificate
3. Replace the bad one
4. Test

Determining which certificate needed to be replaced was a matter of selecting the bad certificate (DigiCert) from the dialog shown above and clicking View Certificate.

This told us that the certificate we needed was “DigiCert Global Root CA” with an expiration date of 11/9/2031.

A quick Google search for “DigiCert Global Root CA” pointed us here: [https://www.digicert.com/digicert-root-certificates.htm](https://www.digicert.com/digicert-root-certificates.htm)

Scrolling down to “DigiCert Global Root CA” showed the expected expiration data, and had a handy Download link.

Now that we had the correct and valid certificate, we just needed to replace the bad one. Here’s how that was done:

1. Click **Start**, click **Start Search**, type **mmc**, and then press ENTER.
2. On the **File** menu, click **Add/Remove Snap-in**.
3. Under **Available snap-ins**, click **Certificates**,and then click **Add**.
4. Under **This snap-in will always manage certificates for**, click **Computer account**, and then click** Next.**
5. Click **Local computer**, and click **Finish**.
6. In the console tree, double-click **Certificates**.
7. Click**Trusted Root Certification Authorities** store.
8. Scroll down to find the bad certificate (**DigiCert Global Root CA**)
9. Right click the bad certificate and click**Delete**
10. This displayed a warning that we could be breaking things, but since we were about to replace it, we clicked**Ok**
11. Right-click the **Trusted Root Certification Authorities** store.
12. Click **Import** to import the certificates and follow the steps in the Certificate Import Wizard, pointing it to the certificate we’d just downloaded from DigiCert.

Once this was done, we launched IE, and visited Outlook Web Access. This time no certificate warnings came up, and Bob finally got to his email.
