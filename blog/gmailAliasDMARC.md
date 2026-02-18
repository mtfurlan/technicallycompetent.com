---
title: Gmail with domain alias emails with DKIM/SPF/DMARC
---

For years, I have been handling email with a gmail account, having my dns
registrar forward all email to my domain to that account, and then adding
sending aliases to the gmail account to send mail.

A few years ago gmail stopped doing the verification by just sending an email
to the email you wanted to send from and started requiring SMTP, but I could
just use an app password and smtp back to that gmail account and it seemed to be
working fine.

Somewhere between 2025-09-12 and 2025-09-23 yahoo started rejecting the emails
because of SPF/DKIM failures.
Looking back, I had seen some warnings about this when I emailed my work
outlook email, or looking at google's original message view.

It turns out I had SPF misalignment because `Return-Path: $gmailUser@gmail.com`
and `From: $domainEmail` didn't match.

Google will mark this as SPF pass but DMARC fail, and in my brief testing it
will usually deliver but sometimes get marked as spam?

The only way forward is to have someone else handle sending email properly.

I ended up choosing [SMTP2GO](https://www.smtp2go.com/) because 1,000
emails/month on their free plain is fine for me.

Steps:
1. create account
2. do the verified senders domain verification setup
3. add smtp user
4. in gmail add a sending alias with the smtp user

----
Update 2026-02-17: I ended up moving to a paid google workspace because calendars kept working poorly with that setup.
