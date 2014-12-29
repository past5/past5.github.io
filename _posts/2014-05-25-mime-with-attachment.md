---
layout: post
title: "MIME Email with Attachment"
tagline: "creating MIME messages with attachment using Linux Shell"
description: "This Linux Shell example demonstrates how to create MIME messages with an attachment"
author: Vsevolod Geraskin
category: [tutorials]
tags: [linux, shell, email]
excerpt: "Sometimes, we may want to have our own MIMEs (as in Mail Extensions messages, although bossing around an entourage of stripy shirt artists may be fun) for use
with such programs as sendmail.  While creating a simple text email using MIME is simple, adding attachments requires a bit of work and can be tricky.  Thankfully, Linux Bash Shell gives us the 
right tools to get the job done."
---
{% include JB/setup %}

#### Yes, we Shell!

Sometimes, we may want to have our own MIMEs (as in Mail Extensions messages, although bossing around an entourage of stripy shirt artists may be fun) for use
with such programs as sendmail.  While creating a simple text email using MIME is simple, adding attachments requires a bit of work and can be tricky.  Thankfully, Linux Bash Shell gives us the 
right tools to get the job done.

In this example, let's say we want to add a .jpg file as an attachment to our MIME message.

#### Step 1 - Specify attachment
We are using the file called [test.jpg](/assets/post_docs/test.jpg) as an attachment.

{% highlight bash %}
[sev@linusaur sendmaildemo]$ attach='test.jpg'
[sev@linusaur sendmaildemo]$ echo $attach
test.jpg
{% endhighlight %}

#### Step 2 - Determine MIME type
Now, we need to determine what MIME file type we are dealing with.  One command we can use is `gnomevfs-info` which gives us the relevant file information.  At the same time, we can use pattern-matching
command such as `awk` to find the specific information we are looking for.

{% highlight bash %}
[sev@linusaur sendmaildemo]$ mimetype=`gnomevfs-info -s $attach | awk '{FS=":"} /MIME type/ {gsub(/^[ \t]+|[ \t]+$/, "",$2); print $2}'`
[sev@linusaur sendmaildemo]$ echo $mimetype
image/jpeg
{% endhighlight %}

#### Step 3 - Encode attachment
MIME requires us to encode the attachment in base-64.  We can use `uuencode` command to do that.  After the encoding, we would need to remove first and last lines, which specify start and end of encoding.
`sed` stream editor will help us easily remove those lines.

{% highlight bash %}
[sev@linusaur sendmaildemo]$ tempfile='attach.temp'
[sev@linusaur sendmaildemo]$ rm -f $tempfile
[sev@linusaur sendmaildemo]$ cat $attach|uuencode --base64 $attach>$tempfile
[sev@linusaur sendmaildemo]$ sed -i -e '1,1d' -e '$d' $tempfile
[sev@linusaur sendmaildemo]$ attachdata=`cat $tempfile`
{% endhighlight %}

At this point, we created [attach.temp](/assets/post_docs/attach.temp) file containing our base-64 encoded jpg attachment.  `$attachdata` variable contains text output of this file.

#### Step 4 - Create email boundary
A multi-part MIME message require boundaries between different parts of the message, and at the start and end of the message body.  Boundary could be anything specified by boundary argument in Content-Type
MIME header.  In our case, we will use the first 32 characters of MD5 checksum of current time in seconds as a message boundary, using `md5sum` command.

{% highlight bash %}
[sev@linusaur sendmaildemo]$ boundary=`date +%s|md5sum`
[sev@linusaur sendmaildemo]$ boundary=${boundary:0:32}
[sev@linusaur sendmaildemo]$ echo $boundary
8360935bdba088f8dc47965c018db705
{% endhighlight %}

#### Step 5 - Create email message
Finally, we can create our MIME [message.mail](/assets/post_docs/message.mail) file and combine it with our boundary and attachment.

{% highlight bash %}
[sev@linusaur sendmaildemo]$ mailfile='message.mail'
[sev@linusaur sendmaildemo]$ rm -f $mailfile

[sev@linusaur sendmaildemo]$ echo "From: senderemail@email.com" >> $mailfile 
[sev@linusaur sendmaildemo]$ echo "To: recipemail@email.com" >> $mailfile 
[sev@linusaur sendmaildemo]$ echo "Reply-To: senderemail@gmail.com" >> $mailfile 
[sev@linusaur sendmaildemo]$ echo "Subject: Test Email" >> $mailfile 
[sev@linusaur sendmaildemo]$ echo "Content-Type: multipart/mixed; boundary=\""$boundary"\"" >> $mailfile 
[sev@linusaur sendmaildemo]$ echo "" >> $mailfile 
[sev@linusaur sendmaildemo]$ echo "This is a MIME formatted message.  If you see this text it means that your" >> $mailfile 
[sev@linusaur sendmaildemo]$ echo "email software does not support MIME formatted messages." >> $mailfile 
[sev@linusaur sendmaildemo]$ echo "" >> $mailfile 
[sev@linusaur sendmaildemo]$ echo "--$boundary" >> $mailfile 
[sev@linusaur sendmaildemo]$ echo "Content-Type: text/plain; charset=ISO-8859-1; format=flowed" >> $mailfile 
[sev@linusaur sendmaildemo]$ echo "Content-Transfer-Encoding: 7bit" >> $mailfile 
[sev@linusaur sendmaildemo]$ echo "Content-Disposition: inline" >> $mailfile
[sev@linusaur sendmaildemo]$ echo "" >> $mailfile
[sev@linusaur sendmaildemo]$ echo "This email was sent with $attach as attachment." >> $mailfile 
[sev@linusaur sendmaildemo]$ echo "" >> $mailfile
[sev@linusaur sendmaildemo]$ echo "" >> $mailfile 
[sev@linusaur sendmaildemo]$ echo "--$boundary" >> $mailfile
[sev@linusaur sendmaildemo]$ echo "Content-Type: $mimetype; name=\"$attach\"" >> $mailfile 
[sev@linusaur sendmaildemo]$ echo "Content-Transfer-Encoding: base64" >> $mailfile 
[sev@linusaur sendmaildemo]$ echo "Content-Disposition: attachment; filename=\"$attach\";" >> $mailfile 
[sev@linusaur sendmaildemo]$ echo "" >> $mailfile 
[sev@linusaur sendmaildemo]$ echo "$attachdata" >> $mailfile
[sev@linusaur sendmaildemo]$ echo "" >> $mailfile 
[sev@linusaur sendmaildemo]$ echo "" >> $mailfile 
[sev@linusaur sendmaildemo]$ echo "--$boundary--" >> $mailfile
[sev@linusaur sendmaildemo]$ echo "" >> $mailfile
[sev@linusaur sendmaildemo]$ echo "" >> $mailfile
{% endhighlight %}

#### Source Code
For a more complete example which uses cron to schedule sendmail to send MIME messages, please check out [my github repository](https://github.com/past5/cron-email).