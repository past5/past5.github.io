---
layout: post
title: "Simple Two Factor Authentication Scheme" 
tagline: "implementing multi-factor server-side authentication the easy way"
description: "This tutorial demonstrate how to use client SSL certificate combined with oauth 2.0 to create a simple two factor authentication scheme."
author: Vsevolod Geraskin
category: [tutorials]
tags: [tls/ssl, security, openssl, http, oauth, shell]
excerpt: "In a perfect world, we would not need any authentication.  In a real world, we often feel protected enough by our `Password1` universal password.  Maintaining sufficient security is often a major pain for
internet organizations, as security and usability are almost always in either or relationship."
---
{% include JB/setup %}

#### Two factors too many
In a perfect world, we would not need any authentication.  In a real world, we often feel protected enough by our `Password1` universal password.  Maintaining sufficient security is often a major pain for
internet organizations, as security and usability are almost always in **either or** relationship.

Unfortunately, oftentimes, the single authentication factor is not enough.  As it happens almost on a daily basis, hackers might steal your social account password and post spam under your account. 
Worse yet, somebody might gain access to your health record and make it public or steal your credit card information from your bank.  Thus, most governments often require software businesses to be 
compliant with multi-factor authentication schemes when it comes to securing sensitive information, such as electronic health records.

Two factor authentication systems can be composed of something a user knows and something a user has.   Also, third authentication factor can be 
something a user is, but, unfortunately, biometrics are beyond the scope of this post.  For simplicity reasons, we will use client certificate as something a user has, and 
username and password as something a user knows.

In this example, let's assume we have an authentication server that provides access to web resources.

#### First Authentication Factor - SSL client certificate
One way to issue a client certificate from our certificate authority is by using the OpenSSL library and the steps below:

**1. Generate RSA private key _testclient.pem_ with 2048-bit encryption strength**

{% highlight bash %}
[root@linusaur certs]# openssl genrsa -des3 -out testclient.pem 2048
Generating RSA private key, 2048 bit long modulus
........................+++
............+++
e is 65537 (0x10001)
Enter pass phrase for testclient.pem:
Verifying - Enter pass phrase for testclient.pem:
{% endhighlight %}

 **2. For self-signed certificate, issue certificate signing request _testclient.csr_ with your details**
 
 {% highlight bash %}
 
[root@linusaur certs]# openssl req -new -key testclient.pem -out
testclient.csr
Enter pass phrase for testclient.pem:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CA
State or Province Name (full name) []:British Columbia
Locality Name (eg, city) [Default City]:Burnaby
Organization Name (eg, company) [Default Company Ltd]:Test Company
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your servers hostname) []:Test
Email Address []:yourname@gmail.com
Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:

{% endhighlight %}

 **3. Issue client certificate _testclient.crt_ against our certificate authority _ariCA.crt_ with 360 days expiry**
 
{% highlight bash %}
[root@linusaur certs]# openssl ca -days 360 -in testclient.csr -out testclient.crt
-keyfile arikey.pem -cert ariCA.crt -policy policy_anything
Using configuration from /etc/pki/tls/openssl.cnf
Enter pass phrase for arikey.pem:
Check that the request matches the signature
Signature ok
Certificate Details:
Serial Number: 5 (0x5)
Validity
Not Before: Jan 13 19:15:05 2014 GMT
Not After : Jan 8 19:15:05 2015 GMT
Subject:
countryName = CA
stateOrProvinceName = British Columbia
localityName = Burnaby
organizationName = Test Company
commonName = Test
emailAddress = yourname@gmail.com
X509v3 extensions:
X509v3 Basic Constraints:
CA:FALSE
Netscape Comment:
OpenSSL Generated Certificate
X509v3 Subject Key Identifier:
3D:09:19:13:66:B6:E7:70:04:24:FE:2A:BC:A4:9F:DC:6F:69:FE:38
X509v3 Authority Key Identifier:
keyid:04:3F:2D:FF:71:44:F0:08:E6:CD:71:CD:81:D5:A7:36:EB:70:96:D2
Certificate is to be certified until Jan 8 19:15:05 2015 GMT (360 days)
Sign the certificate? [y/n]:y
1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
{% endhighlight %}

Now, we can give _testclient.crt_ to a user which they will use as the first authentication factor.  Every time the user tries to access a web resource on our server, our authentication server should 
request this client certificate, check it against our certificate authority, and either grant or deny access.

#### Second authentication factor - username and password

Once a user passes the first authentication step, the server should then validate their username and password.  Arguably, the simplest way for a user software to pass username and password to our web 
service would be using HTTP protocol [basic authentication scheme](http://tools.ietf.org/html/rfc2617), 
which uses base-64 encrypted username and password.  For instance, user HTTP request header to our authentication server could look like this:

{% highlight HTTP %}
POST /authorize HTTP/1.1
Authorization: Basic QkMwMDAwMEMzNTpTZXZDbGllbnRUZXN0
{% endhighlight %}

Where `QkMwMDAwMEMzNTpTZXZDbGllbnRUZXN0` would be the output of a client-side function such as `string base64encrypt(username + ":" + password)`.

Upon successful validation of username and password, our authentication server could issue a token which expires after a certain time, with which a client finally could access various resources. 
Using the token we issued, the authenticated user resource request headers could look like this:

{% highlight HTTP %}
POST /ResourceLocation HTTP/1.1
Authorization: Bearer SN3528Ev0r~lkMTq2jY7
{% endhighlight %}

Where `SN3528Ev0r~lkMTq2jY7` is a token issued by our server of type Bearer.  For more information on Bearer Tokens and Oauth 2.0 authentication, please check 
[The OAuth 2.0 Authorization Framework](http://tools.ietf.org/html/rfc6749) and [The OAuth 2.0 Authorization Framework: Bearer Token Usage](http://tools.ietf.org/html/rfc6750).

#### Both authentication factors working together

<img class="float-right" width="480pt" src="/assets/post_images/twofactor1.png" alt="System Diagram of Two Factor Authentication System" />
The system diagram demonstrates issuing tokens by a two factor authentication system, where usernames, passwords, and access tokens are stored in MySQL database.  The authentication protocol flow is
described below:

1. Web resource client initiates SSL handshake and provides their SSL certificate to the server.
2. Authentication server verifies client certificate against root certificate authority stored on the server.
3. Client sends an HTTP(POST) request to authentication server with HTTP basic authorization header (base64-encoded username:password).
4. Authentication server returns a bearer access token or HTTP error (ie. 401 Unauthorized).
5. Clients sends an HTTP(POST) request to resource server containing bearer token in authorization header.
6. Resource server verifies an access token and responds with a requested resource. 

