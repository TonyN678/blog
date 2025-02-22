+++
date = '2024-12-21T14:27:51+11:00'
title = 'My Own Email Domain'
toc = true
+++
# How to Set Up Your Own Email on Your Domain Name

Having your own email address at your custom domain name adds a professional touch to your communications. This guide will walk you through the process of setting up your own email address using your domain and integrating it with Gmail for sending and receiving emails. This guide will show how I set up my own email address at tien@tienng.xyz.

I wrote this guide using informations from [IdeaSpot's video](https://www.youtube.com/watch?v=nNGcvz1Sc_8&list=WL&index=12) so feel free to check it out.
## Prerequisites
- A custom domain name (e.g., `yourdomain.com`).
- Access to your domain's DNS settings.
- A transactional email service like [Brevo](https://www.brevo.com/).
- A Cloudflare account with 'yourdomain.com' already linked.
- A Gmail account.

---

## Step 1: Receiving Emails with Cloudflare Email Routing
Cloudflare offers a free email routing service that allows you to receive emails sent to your custom domain and forward them to an existing email address (e.g., Gmail).

### Configure Cloudflare Email Routing
1. Log in to your Cloudflare account and select your domain.
2. Navigate to the **Email** tab in the Cloudflare dashboard.
3. Click **Set up Email Routing** and follow the prompts.
4. Add the required DNS records to enable email routing:
   - **MX Record**: Points to Cloudflare’s email servers (e.g., `mx.example.com`).
   - **TXT Record**: For SPF, to authorise email forwarding from your domain.
   - **CNAME Record**: For DKIM, to ensure email authentication.

   Cloudflare will provide specific instructions for adding these records.

5. Configure email forwarding by specifying the custom email address you want to create (e.g., `you@yourdomain.com`) and the destination email address (e.g., `yourname@gmail.com`).
6. Save your changes and allow time for the DNS records to propagate (typically a few minutes to a few hours).

### Test Email Forwarding
- Send a test email to your custom email address (e.g., `you@yourdomain.com`).
- Confirm that the email is successfully forwarded to your Gmail inbox.

---

## Step 2: Sending Emails with Your Custom Domain
To send emails from your custom email address, you’ll need to set up an SMTP service. In this guide, we’ll use Brevo.

You could choose between gmail SMTP or Brevo SMTP; If choose gmail SMTP, skip anything related to Brevo.

### Configure Your Domain in Brevo
1. Sign up for a Brevo account if you don’t already have one.
2. Click on Profile name, go to "Senders, Domains & Dedicated IPs", go to "Domains" tab, add your custom domain.
3. Brevo will help you with adding DNS records to your domain, since I linked my domain name to Cloudflare, I could just choose "sign-in with Cloudflare" option and Brevo would add the DNS records automatically for me.
4. Once the DNS records propagate (this may take a few hours), your domain will be "Authenticated".
   Brevo provides detailed instructions for adding these records in your DNS settings.
5. Then go to "SMTP & API" and generate a new SMTP key and remember the password.

### Connect Your Custom Email to Gmail for Sending
1. Open Gmail and go to **Settings** (gear icon) > **See all settings**.
2. Navigate to the **Accounts and Import** tab.
3. Under **Send mail as**, click **Add another email address**.
4. Enter your name (sender's name) and custom email address (e.g., `you@yourdomain.com`) which you set-up earlier in Cloudflare, then click **Next Step**.
5. Enable "alias" to easily choosing which email to send from in every mail, if not enabled, gmail will send all email from your custom email address.
#### For Brevo SMTP 
5. Use the following SMTP settings:
   - **SMTP Server**: `smtp-relay.brevo.com`
   - **Port**: `587`
   - **Username**: Your Brevo login email (In the SMTP & API tab)
   - **Password**: The SMTP key you generated earlier
   - **Authentication**: Ensure "Secured connection using TLS" is selected

#### For Gmail SMTP  
5. Use the following SMTP settings:
   - **SMTP Server**: `smtp.gmail.com`
   - **Port**: `587`
   - **Username**: Your gmail (The gmail you forwarded to)
   - **Password**: [App Password generated by Google](https://support.google.com/mail/answer/185833?hl=en)
   - **Authentication**: Ensure "Secured connection using TLS" is selected
6. Click **Add Account** and verify ownership by entering the code sent to your custom email address (check your Gmail inbox for the forwarded message)

### Troubleshooting Sending Issues
If you encounter an "Authentication failed" error:
1. Double-check the SMTP settings in Gmail and ensure they match Brevo’s specifications.
2. Verify that your SMTP key is active and correctly entered.
3. Consult this [Brevo Community post](https://community.brevo.com/t/fixed-authentication-failed-when-connect-with-gmail/1933/3) for setting up Gmail SMTP instead.

---

## Final Steps: Test Your Setup
1. Send a test email from Gmail using your custom email address.
2. Verify that you can send and receive emails without issues.

Congratulations! You’ve successfully set up your custom email address for both sending and receiving emails.
