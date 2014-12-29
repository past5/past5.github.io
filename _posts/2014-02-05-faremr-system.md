---
layout: post
title: "FAR-EMR System"
tagline: "allowing healthcare professionals in British Columbia securely exchange data in the form of clinical documents"
description: "FAR-EMR project delivers a proof-of-concept system which allows healthcare professionals to exchange clinical documents containing patient medication summaries. The system is
composed of authentication server, resource server, database, and web service clients.  FAR-EMR is written in C using OPENSSL and libxml2 libriaries and stores data in MYSQL database."
author: Vsevolod Geraskin
category: projects
tags: [c, mysql, tls/ssl, rest, http, xml, agile, tcp/ip, security, oauth]
excerpt: "Currently, various healthcare providers in our province do not have immediate access to a
patient's health data upon receiving the patient into their care due to their technological or funding limitations. For example, doctors or
emergency ambulance teams cannot promptly view patient medication history, which would
allow them to make informed decisions on patient's treatment. This medication history is
available on the Pharmanet and is accessible by Patient's pharmacist or nurse. The
challenge is to get appropriate data from one healthcare professional to another in a prompt and simple
manner, while conforming to various security and messaging standards."
---
{% include JB/setup %}

#### Problem Statement
Currently, various healthcare providers in our province do not have immediate access to a
patient's health data upon receiving the patient into their care due to their technological or funding limitations. For example, doctors or
emergency ambulance teams cannot promptly view patient medication history, which would
allow them to make informed decisions on patient's treatment. This medication history is
available on the Pharmanet and is accessible by Patient's pharmacist or nurse. The
challenge is to get appropriate data from one healthcare professional to another in a prompt and simple
manner, while conforming to various security and messaging standards.

Due to sensitivity of health data, government requires online healthcare systems to use two
factor authentication. Therefore, in addition to usernames and passwords, the system must
also have a secondary unrelated method of authenticating users. For example, two factor
authentication can be something a user knows, such as a password, and something a user
has, such as a client SSL certificate.

Finally, the exchange of healthcare data between various system is subject to health
messaging standards. In this case, the data is exchanged in the form of clinical documents
created by a pharmacist or a nurse and viewed by a doctor or emergency medical technician.
The clinical document is an HL7 v3 XML file which contains base-64 encoded PDF
segment containing patient health data.

#### Project Description
FAR-EMR system is a secure and scalable multi-part server I built as an alternative to costly EMR systems in order to allow clinical document exchange between healthcare professionals.
The type of a clinical document the system is created to handle contains patient's medication history and is created in pharmacy or
nursing home by a pharmacist or a nurse for use by doctors or other relevant healthcare
professionals.

The total project effort was approximately 1500 hours, and was sponsored by my main client, Applied Robotics Inc.  FAR-EMR project used Agile development methodology, while closely following XP process.
The small size of project team and close collaboration with nursing homes, pharmacies
and various EMR vendors made Agile the appropriate methodology to develop a highly
appealing end product with a focus on healthcare professionals and their EMR applications. 

FAR-EMR system is composed of authentication server, resource server, database, and web service clients, which
implement the following functionality:

- **authentication server** uses two factor authentication method in the form of user credentials and SSL client certificate to grant users access to the resource server;
- **resource server** provides a set of secure RESTful webservices allowing users to send and receive clinical documents, validate document structure, and create patients and practitioners;
- **database** provides tables to store user information, patients, practitioners, and clinical
- **web service client** allows pharmacy software to send clinical documents to the resource server from a command line.

<img class="float-left" width="480pt" src="/assets/post_images/faremr1.png" alt="Context Diagram of FAR-EMR System" />

Upon receiving a patient into their care, a health professional contacts their colleague to
request a patient's health information. Then, the recipient of a call generates a medication
history report in their health system. After the report is complete and verified, the supplier's
health system invokes a command line client. Command-line client connects to FAR-EMR
HTTPS Authentication server with the provided SSL client certificate and supplier's username
and password and receives a token upon successful authentication. Finally, the command
line client connects to the resource server and submits the received token along with XML
clinical document data. The resource server verifies the token and stores the data into
MySQL database.

Once the data is available in the database, the health professional requesting health data
connects to authentication server using their miscellaneous application and sends
authentication credentials. The server then goes through the same authentication process
described above and issues a token. Thus, using the token, a consumer application can
request and receive clinical data from resource server.

Technologies used on this project are as follows:

- **MySQL database**: FAR-EMR uses MySQL running on Linux server to store data. The database contains
tables for storing Clinical Documents, Patients, Practitioners, and User Authentication
information.

- **C Language**: FAR-EMR lightweight authentication and resource servers are written entirely in C language.

- **OpenSSL Library**: FAR-EMR uses OpenSSL library to create certificates, validate clients during first stage of
authentication, and encrypt all communication between client and server.

- **Libxml2 Library**: FAR-EMR uses Libxml2 to parse XML data and create XML responses.

#### Project Outcomes
FAR-EMR system satisfied project sponsor's requirements: during acceptance testing, we were able to handle hundreds of simultaneous clinical document submissions and requests on a single laptop. 
The system is also a demonstration of a functioning scalable and
secure EMR server without resorting to feature-rich, but difficult to configure and
secure, web frameworks and servers. Furthermore, due to its event-driven architecture, FAR-EMR
scales far better than EMRs deployed on process-driven servers such as Apache. While further
development work is required to make FAR-EMR production ready, the proof-of-concept
demonstration was a success, and the system could present a viable alternative
to existing EMR products in the near future.

The XP/Agile principles were fulfilled as follows:

- constantly refining user requirements in the form of user stories;
- recognizing changing FAR-EMR requirements, and redesigning the product accordingly;
- keeping the development process simple in the form of weekly iterations;
- and using completed software demonstrations to project sponsor as the true measure of progress.

#### Testimonial
_We now have a working internal system and are integrating that capability with the resources of a leading provider of laboratory results in BC to offer very soon a production 
version of a complete electronic patient health record...  ...I am confident that the results derived from this [FAR-EMR] project will form the foundation of a significant advance in the distribution of health information in our province and perhaps 
the rest of the country._

- Dennis B., President, Applied Robotics Inc.
