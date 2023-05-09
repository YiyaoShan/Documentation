---
title: e-Box Enterprise via Webservice
---

You don't want to check your messages one by one anymore? Become a Document Consumer. You can read all the messages you have received in your e-Box Enterprise with **a single click** via a **RESTful Webservice**. Follow the onboarding self-service process below to integrate as a Document Consumer of the e-Box Enterprise. The Onboarding can be done in both Acceptance and Production environments. 

![Diagram DocConsumer Onboarding Process](https://github.com/YiyaoShan/Documentation/blob/main/DocConsumer%20Onboarding%20Processus.png)


# Onboarding Process to become a Document Consumer (DocConsumer)
You’ve not yet used e-Box Enterprise? Before you can become a DocConsumer of e-Box Enterprise, you must create an account and do security settings. 
You already use the e-Box Enterprise? Go directly to the section ***1.2 Accept Terms of Use***.



## Step 1: Onboarding self-service DocConsumer
### 1.1 User account management
In order to become a DocConsumer, you need an **Access Manager (AM)** or a **Co-Access Manager (Co-AM)** to activate and manage your technical "Webservice" channel. Follow the [Procedure for Access Management in UMan.pdf](https://www.socialsecurity.be/site_fr/general/helpcentre/rest/documents/pdf/procedure_pour_gestion_des_acces_UMan_FR.pdf) manual to designate an AM/co-AM and manage user access.

### 1.2 Accepting the terms of use
Currently, the activation of the WebService channel is done in "all or nothing" mode: your company has access not only to the Webservice of e-Box Enterprise, but also to [other services within the Social Security](https://www.socialsecurity.be/site_fr/employer/infos/online-services.htm.). Moreover, the access is **always** in the name of your company. The responsibility remains with the company in case of misuse of the service / credentials. For the security of the connections, your company is obliged to accept the terms of use for DocConsumer.

### 1.3 Creating a technical user
The GA/co-GA should now create a technical user in the "[Access Management](https://www.socialsecurity.be/site_fr/employer/applics/umoe/index.htm)" service. To do this, consult the manual [Create your Webservice channel on the Social Security portal.pdf](https://www.socialsecurity.be/site_fr/general/helpcentre/rest/documents/pdf/webservices_creer_le_canal_FR.pdf).
The certificate to be used must be a **X.509 certificate**. Any official issuer is acceptable. Please avoid using a self-signed certificate. Make sure you follow the [expected format](https://dev.eboxenterprise.be/docs/common/x509_certificate). It is also important to have **a separate certificate for each environment** (acceptance, production).

*P.S. Gestionnaire  Local (GL)=Access Manager (AM); co-Gestionnaire Local (co-GL)=co-Access Manager (co-AM)*

### 1.4 Activating a Webservice channel
Once the technical user has been created, your AM/co-AM must activate the "Webservice" channel in the "[Access Management](https://www.socialsecurity.be/site_fr/employer/applics/umoe/index.htm)" service. Refer to the manual [Add your Webservice channel on the Social Security portal.pdf](https://www.socialsecurity.be/site_fr/general/helpcentre/rest/documents/pdf/webservices_ajouter_le_canal_FR.pdf) to activate the channel.



## Step 2: OAuth configuration
### 2.1 Company registration in OAuth Server
The previous step involves the registration of your company (identified by its BCE-KBO number) in the Social Security OAuth authorization server, thanks to which you get an **OAuth client-id** that is associated with your company and your certificate, allowing you to generate an **OAuth AccessToken** which serves for consulting the e-Box Enterprise RESTful API. 

### 2.2 Retrieving the OAuth AccessToken
The Social Security’s RESTful Webservice is highly secured. Before you can access to the e-Box Enterprise RESTful Webservice, you must request an OAuth AccessToken from the OAuth authorization server. 
The [OAuth introspect](https://github.com/e-Box-Enterprise-Belgium/examples/tree/master/ouath-introspect) example shows how an OAuth AccessToken can be retrieved. You must use your certificate to authenticate your company. The ``GetAccessTokenV5.getAccessToken()`` method is responsible for obtaining the token.

<table>
<tr><td>OAuth Authorization Server URL (ACC)</td><td>https://services-acpt.socialsecurity.be/REST/oauth/v5/token</td></tr>
<tr><td>Audience (ACC)</td><td>https://services-acpt.socialsecurity.be/REST/oauth/v5/token</td></tr>
<tr><td>OAuth Authorization Server URL (PRD)</td><td>https://services.socialsecurity.be/REST/oauth/v5/token</td></tr>
<tr><td>Audience (PRD)</td><td>https://services.socialsecurity.be/REST/oauth/v5/token</td></tr>
</table>

As a Document Consumer, your AccessToken will contain the necessary scopes to perform all possible requests concerning your e-Box:
- **summary_own_ebox** (``scope:document:management:consult:ws-eboxrestentreprise:summaryownebox``) to get the summary of your e-Box;
- **documentconsumer** (``scope:document:management:consult:ws-eboxrestentreprise:documentconsumer``) to get and perform authorized actions on all messages of your e-Box ;
- **reference_data** (``scope:document:management:consult:ws-eboxrestentreprise:referencedata``) to retrieve details of messageTypes, senderOrganizations and senderApplications.
- **Provider Registry** (``scope:documentmanagement:ebox:enterprise:federation-rest:registry``) to get the list of Document Providers.

More info :
- [OAuth2 Integration Client Credential.pdf](https://www.socialsecurity.be/site_fr/general/helpcentre/rest/documents/pdf/doc_portal_oauth2_client_credential_FR.pdf)
- Java code to implement OAuth : [ClientCredentialTokenGenerator.java](https://www.socialsecurity.be/site_fr/general/helpcentre/rest/documents/ClientCredentialTokenGenerator.java)



## Step 3: e-Box Enterprise consultation via REST Webservice
### 3.1 Find the Document Providers
Once you have obtained your token, you can find the list of Document Providers by making a GET request to the Provider Registry (don't forget to use the AccessToken retrieved in the previous step):
| Environment| URL Provider Registry                                                                     |
|------------|------------------------------------------------------------------------------------------------|
| Acceptance | ``https://services-acpt.socialsecurity.be/REST/ebox/enterprise/federation/v1/messageProviders``|
| Production | ``https://services.socialsecurity.be/REST/ebox/enterprise/federation/v1/messageProviders``     |

In the response, you will find our Document Provider:

| Environment| Endpoint e-Box enterprise                                                           |
|------------|-------------------------------------------------------------------------------------|
| Acceptance | ``https://services-acpt.socialsecurity.be/REST/ebox/enterprise/messageRegistry/v2/``|
| Production | ``https://services.socialsecurity.be/REST/ebox/enterprise/messageRegistry/v2/``      |

### 3.2 Consulting your e-Box Enterprise via REST Webservice
The consultation of your e-Box is done via an HTTP ***GET*** through all the methods that start with ```/ebox``` defined in [MESSAGE REGISTRY SERVICE](https://dev.eboxenterprise.be/docs/spec/specifications)  of e-Box. This specification is available in ``.yaml`` format. Thus, a DocConsumer can not only view his messages, but also the ReferenceData. The methods in the *"publication"* section are only available for the Document Sender and the Document Provider.

Some points of attention:
- Messages will be considered as read when the main content has been retrieved. On the e-Box Enterprise frontend, the visual distinction is the bolding ("unread") or not ("read") of the message title.
- Messages retrieved by DocConsumer remain available in the e-Box until their expiration date. 
- The e-Box Enterprise application (user interface version) is always available (and must be used in case of errors with the use of technical interfaces).




# The RESTful Webservice for e-Box Enterprise
The RESTful Webservice for e-Box Enterprise offers you several functionalities: 
1. **Messages** : You can not only view all messages in your e-Box by simply calling ```/messages```, but also filter messages by expiration date, sender, [partition](https://dev.eboxenterprise.be/docs/federation/partition), etc. As on the user interface, you can also change the visibility and the partition of messages. 
   - *Partition*: Partitions are "sub-boxes" of the e-Box that have their own identity and their own user access management. They allow to 'classify' the messages and thus each user receives only the messages of his partition instead of all the messages received by his company. This improves security in case of confidential messages and helps to organize the received messages. DocConsumers are not limited to partitions as they can view all messages in the e-Box without applying any filter.
   - *Message forAction*: Some messages require an action from the user. (e.g., print a document) As a DocConsumer, once you have completed the action, you can "execute" the message action by making a "*patch*" via e-Box Enterprise REST Webservice.
   - *ReferenceData*: it allows you to retrieve the details of messageTypes, senderOrganizations and senderApplications.
   - *Statistics*: it tells you how many messages are about to expire, how many messages require action on your part and how many messages are registered messages.
   - *Reply*: you can reply to messages (if applicable) and execute the message action when it has been done.  

# Need more info? 
- [What is e-Box Enterprise](https://wwwacc.eboxenterprise.be/fr/index.html) 
- [What is a partition](https://dev.eboxenterprise.be/docs/federation/partition)
- [e-Box Enterprise Services Specifications](https://dev.eboxenterprise.be/docs/spec/specifications)
