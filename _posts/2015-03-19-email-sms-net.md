---
layout: post
title: "Email SMS Message in .NET"
tagline: "sending SMS message using .NET SmtpClient and mobile carrier mail-to-SMS service"
description: "This tutorial shows how to quickly send SMS message sending SMS message using .NET SmtpClient and mobile carrier mail-to-SMS service."
author: Vsevolod Geraskin
category: [tutorials]
tags: [code, .net, smtp, email, sms, c#]
excerpt: "Our app needs to send notifications to customers' mobiles, but we don't have much time.  Fortunately, each mobile carrier provides email-to-SMS service for clients having a data package. 
Thus, the easiest way to send an SMS message to a cell phone with a data plan would be to compose an email message and send it to the provided address. "
---
{% include JB/setup %}

<img class="float-right" width="300pt" src="/assets/post_images/sms1.jpg" alt="SMS" />

#### Email to SMS
Our app needs to send notifications to customers' mobiles, but we don't have much time.  Fortunately, each mobile carrier provides email-to-SMS service for clients having a data package. 
Thus, the easiest way to send an SMS message to a cell phone with a data plan would be to compose an email message and send it to the provided address.   

#### Carrier services
Each carrier provides an email-to-text service in the format 6041234567@carrier.ca.  All we need to do is to compose an email message containing our text and send it to that email address.
Most of the carriers in Canada are listed below:

- Bell/President's Choice/Solo: `<phonenumber>@txt.bellmobility.ca` or `<phonenumber>@txt.bell.ca`
- Fido/Microcell: `<phonenumber>@fido.ca`
- Rogers/Chatr: `<phonenumber>@pcs.rogers.com`
- Telus: `<phonenumber>@msg.telus.com`
- Virgin Mobile Canada: `<phonenumber>@vmobile.ca`
- Koodo: `<phonenumber>@msg.koodomobile.com`

#### So how can we determine the carrier?
We can't.  In this solution, we simply send an email to ourselves while BCCing all the carriers.  Only the appropriate carrier which has the recipient's phone number will deliver the message.

#### Source?
Of course!  In the example below, we use GMail's SMTP service and .NET SmtpClient to send an email message. 
 

{% highlight C# %}
		private void SendMail(String phoneNumber, String textMessage) {
			
            MailMessage smsMessage = new MailMessage();
            
			//specifying our gmail account
            smsMessage.From = new MailAddress("testaccount@gmail.com");
            smsMessage.To.Add(new MailAddress("testaccount@gmail.com"));
			
			//adding all the carriers
            smsMessage.Bcc.Add(new MailAddress(phoneNumber + "@txt.bellmobility.ca"));
            smsMessage.Bcc.Add(new MailAddress(phoneNumber + "@txt.bell.ca"));
            smsMessage.Bcc.Add(new MailAddress(phoneNumber + "@fido.ca"));
            smsMessage.Bcc.Add(new MailAddress(phoneNumber + "@pcs.rogers.com"));
            smsMessage.Bcc.Add(new MailAddress(phoneNumber + "@msg.telus.com"));
            smsMessage.Bcc.Add(new MailAddress(phoneNumber + "@vmobile.ca"));
            smsMessage.Bcc.Add(new MailAddress(phoneNumber + "@msg.koodomobile.com"));

            smsMessage.Body = textMessage;
			
			//specifying gmail settings
            SmtpClient sevGmailClient = new SmtpClient {
                Host = "smtp.gmail.com",
                Port = 587,
                EnableSsl = true,
                UseDefaultCredentials = false,
                Credentials = new System.Net.NetworkCredential("yourusername", "yourpassword")             
            };

           try {
           		//sending email message
                sevGmailClient.Send(smsMessage);
           }
           catch (Exception e) {
               //catching an error 
           }
        }
{% endhighlight %}

That's it!  Simple, and sometimes useful.













 




      



  










