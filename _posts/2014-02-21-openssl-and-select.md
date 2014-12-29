---
layout: post
title: "OpenSSL and select()"
description: "This tutorial shows how to correctly read OpenSSL records within C select() statement."
tagline: "reading OpenSSL records correctly within C select() statement"
author: Vsevolod Geraskin
category: [tutorials]
tags: [c, tls/ssl, security, openssl, select, code]
excerpt: "In recent years, many event-driven server-side technologies, such as Node.js javascript library and lighttpd web server, enabled programmers to easily build applications which scale much better 
than traditional span new process or thread for each new connection servers, such as Apache.  Both node.js javascript engine and lighttpd are written in C/C++ language using I/O multiplexors
which have been in existence for many years.  One of such multiplexors is C `select()` statement which has been around forever.  Securing software communication channels is one of the major issues 
in today's world, and,  in this tutorial, we will examine how to read OpenSSL records using `select()` multiplexor, should we decide to create our own secure event-driven library or the hot new
web server from ground up."
---
{% include JB/setup %}

#### When old becomes new
In recent years, many _event-driven_ server-side technologies, such as Node.js javascript library and lighttpd web server, enabled programmers to easily build applications which scale much better 
than traditional _span new process or thread for each new connection_ servers, such as Apache.  Both node.js javascript engine and lighttpd are written in C/C++ language using I/O multiplexors
which have been in existence for many years.  One of such multiplexors is C `select()` statement which has been around forever.  Securing software communication channels is one of the major issues 
in today's world, and,  in this tutorial, we will examine how to read OpenSSL records using `select()` multiplexor, should we decide to create our own secure event-driven library or the hot new
web server from ground up.

#### Traditional select() loop

{% highlight C %}
while (TRUE) {
	temp_set = master_set;               // structure assignment
   		
	if ((select_result = select(max_fd_number + 1, &temp_set, NULL, NULL, NULL)) == -1) fatal("ERROR: select error\n");

	if (FD_ISSET(socket_id, &temp_set)) {
		// accept new client connection
		client_len = sizeof(client);
		this_socket_id = accept(socket_id, (struct sockaddr *) &client, &client_len);
	} else {
		// check clients for data
		for (i = 0; i <= client_index; i++)	{
			if ((socket_fd = clients[i]) < 0) continue;
				
			if (FD_ISSET(socket_fd, &temp_set)) {
				//read data	
						
				if (--select_result <= 0) break;        // no more readable descriptors
			}
		}
	}
   
	FD_SET (this_socket_id, &master_set);     // add new descriptor to set

	if (this_socket_id > max_fd_number) max_fd_number = this_socket_id;	// add to select
	if (i > client_index) client_index = i;	// new max index in client[] array
	if (--select_result <= 0) continue;	// no more readable descriptors
}
{% endhighlight %}

The above code shows one way to implement a `select()` read on non-encrypted socket connection. Basically, what happens above is we run select() statement, check if the file descriptor ready to be read in the 
select set is a listening socket or an existing connected socket, and either accept new connection or read data on the existing connection.  The actual read data operation is simple: for instance, we can 
read bytes into `buffer_array` from socket using `read` statement as shown below.

{% highlight C %}
if (FD_ISSET(socket_fd, &temp_set)) {
	//read data	
	if ((bytes_read = read(socket_fd, buffer_array, BUFFER_LENGTH)) > 0) { ... }
}
{% endhighlight %}

#### OpenSSL can of worms
Let's say we had to secure the above connection.  We would probably use OpenSSL library and use `SSL_read` statement instead to read and decrypt SSL record.  Our read data operation could look like:

{% highlight C %}
if (FD_ISSET(socket_fd, &temp_set)) {
	//read data	
	if ((bytes_read = SSL_read(SSL, buffer_array, BUFFER_LENGTH)) > 0) { ... }
}
{% endhighlight %}

But implementing the above code, `select()` could actually mislead the application and signal that SSL data is ready to read when, in fact, it is not ready.  Why is that?  
Because `select()` is solely concerned with the contents of the encrypted network buffer, while the `SSL_read` reads the SSL buffer into which OpenSSL places the decrypted data. 
`SSL_read` has to read the entire SSL record in order to deliver the first decrypted byte to the program, while `select()` returns immediately after the first encrypted byte is ready, 
creating the following situation:

1. First `select()` call returns signalling encrypted data is ready to read.
2. `SSL_read` empties network buffer, but is not ready to deliver the decrypted data in the SSL buffer to the application due to decryption overhead.
3. Consecutive `select()` call does not return since network buffer is empty.

#### And the solution
We need another SSL read loop to make `select()` wait until SSL record is completely placed into SSL buffer and decrypted data becomes available:

{% highlight C %}
if (FD_ISSET(socket_fd, &temp_set)) {
	//read data	
	do  {
		read_blocked = 0;
		bytes_read = SSL_read(SSL,buffer_array,BUFFER_LENGTH);;

		//check SSL errors
		switch(ssl_error = SSL_get_error(SSL,bytes_read)){
			case SSL_ERROR_NONE:
				//do our stuff with buffer_array here
			break;
			
			case SSL_ERROR_ZERO_RETURN:		
				//connection closed by client, clean up
			break;
			
			case SSL_ERROR_WANT_READ:
				//the operation did not complete, block the read
				read_blocked = 1;
			break;
			
			case SSL_ERROR_WANT_WRITE:
				//the operation did not complete
			break;
			
			case SSL_ERROR_SYSCALL:
				//some I/O error occured (could be caused by false start in Chrome for instance), disconnect the client and clean up
			break;
							
			default:
				//some other error, clean up
				
	} while (SSL_pending(SSL) && !read_blocked);
}
{% endhighlight %}

And Voila! In the above code, we obtain number of readable bytes buffered in an SSL object through `SSL_pending` function, and loop while there is still bytes to read.   This loop makes select() wait 
until we have the complete decrypted SSL record.  For a more complete description, please read this excellent RTFM Inc. paper [part 1](/assets/post_docs/openssl1.pdf) and
[part 2](/assets/post_docs/openssl2.pdf) by Eric Rescorla which helped me solve this particular problem.

#### Activity diagram of SSL sockets + select()

<img width="480pt" class="resize" src="/assets/post_images/select1.png" alt="Activity Diagram of SSL sockets + select()" />