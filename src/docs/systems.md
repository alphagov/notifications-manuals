# GOV.UK Notify Systems Overview


### System Context/Boundary
![Boundry](https://drive.google.com/uc?id=0B4iP55OVXryoSDZnVlIwdlJxWU0)

Notify provides a service for Government Departments to send notifications to their end users. Hence, our 'users' are usually the Government departments rather than the end users. We provide 3 type of services: email, text messaging and letters (available soon).  

This system context/boundary diagram shows where external users and systems interact with Notify's backend systems. The interaction points are as follows:

### Left-hand side/ Inbound interactions
* Users use Notify's **admin** console webpages, which interacts with Notify backend systems via REST-API. The admin console can be used to 
    - upload and/or schedule csv jobs
    - send test notifications
    - view outbound and inbound sms messages sent
    - change settings 
    - view billing and status information. 
* Users can programmatically interacts with our system via REST-API for uploading jobs for all **email**, **sms** and **letter** services. 

### Right-hand side/ Outbound Interactions
Notify provides a service for Government Departments to send notifications to their end users. Hence, our 'users' are usually the Government departments rather than the end users. We provide 3 type of services: email, text messaging and letters (available soon).
This system context/boundary diagram shows where external users and systems interact with Notify's backend systems. The interaction points are as follow:


* Notify interacts bi-directionaly with our SMS providers via REST-API. The SMS providers provide **text messages** services for our users to send messages to their end users. We provide **inbound sms** service for our end-users to reply messages. 
* Notify interacts with our **email** service provider via AWS SES and SNS. The status of emails are sent back to our system via REST API
* Notify interacts with our AWS FTP server via boto3 and REST API, which allows DVLA (our letter service provider) to send and receive documents using ftp, for **letter services**. DVLA currently handles the posting of the letters. 


