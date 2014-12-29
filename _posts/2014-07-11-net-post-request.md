---
layout: post
title: ".NET POST Request"
tagline: "crafting .NET POST request using HTTPWebRequest object"
description: ".NET tutorial which shows how to use HTTPWebRequest object in order to quickly create a manual POST request"
author: Vsevolod Geraskin
category: [tutorials]
tags: [.net, http, code]
excerpt: "Oftentimes, such as when hacking demos, we want our client side app talking to some form backend without spending too much time on it. One way to do it in a Microsoft environment is to do a quick 
POST request using `HTTPWebRequest` object.  `HTTPWebRequest` does exactly what the name suggests and saves us a lot of time from having to write our own HTTP calls."
---
{% include JB/setup %}

#### Demo it now
Oftentimes, such as when hacking demos, we want our client side app talking to some form backend without spending too much time on it. One way to do it in a Microsoft environment is to do a quick 
POST request using `HTTPWebRequest` object.  `HTTPWebRequest` does exactly what the name suggests and saves us a lot of time from having to write our own HTTP calls.

#### As simple as URL
During the first step, we would split our URL such as `http://www.someserver.com/service.aspx?var1=val1..` into the address 
of the server and the body containing our POST variables.  Of course, if we were creating a GET request, then we would not need to do that.

{% highlight C# %}
String url = "http://www.someserver.com/service.aspx?var1=value1&var2=value2";
string [] httpSend = url.Split('?');

Uri uri = new Uri(httpSend[0]);
{% endhighlight %}

Next, we would create `HTTPWebRequest` object using the above uri in the constructor and assign whichever HTTP header variables we need.

{% highlight C# %}
HttpWebRequest request = (HttpWebRequest)WebRequest.Create(uri);
request.AllowAutoRedirect = true;
request.KeepAlive = true;
request.UserAgent = "Past5 demo app";
request.Accept = "*/*";
{% endhighlight %}

Since we are POSTing to some form backend, we would also need to tell the server that we are submitting a form:

{% highlight C# %}
request.ContentType = "application/x-www-form-urlencoded";
request.Method = "POST";
{% endhighlight %}

And finally, since we are sending a POST request, our URL GET variables will go into the body of our HTTP request.

{% highlight C# %}
byte[] content = Encoding.UTF8.GetBytes(httpSend[1]);
request.ContentLength = content.Length;
	
using (Stream stream = request.GetRequestStream()) {
	stream.Write(content, 0, content.Length);
	stream.Close();
}
                
{% endhighlight %}

#### The whole HTTP request
{% highlight HTTP %}
POST http://www.someserver.com/service.aspx HTTP/1.1
User-Agent: Past5 demo app
Accept: */*
Content-Type: application/x-www-form-urlencoded
Host: www.someserver.com
Content-Length: 23
Expect: 100-continue
Connection: Keep-Alive

var1=value1&var2=value2
{% endhighlight %}

#### P.S. And what about the response?
To obtain the response from the server, we would simply use `HTTPWebResponse` object and use StreamReader to get the body of the response.

{% highlight C# %}
HttpWebResponse response = (HttpWebResponse)request.GetResponse();
StreamReader reader = new StreamReader(response.GetResponseStream());
	
if (request.HaveResponse) String responseBody = reader.ReadToEnd();
	
...
{% endhighlight %}