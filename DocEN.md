---
title: e-Box Enterprise via Webservice
---

Does your enterprise receive a large number of documents in your e-Box? Is it difficult for you to process messages manually in your e-Box online? Technical integration with e-Box via a technical interface (web service) can be a solution, enabling messages to be retrieved in "packages" ("Document Consumer" solution).
All your e-Box messages can then be retrieved directly from your internal IT system. This enables you to manage message distribution within your enterprise. You can also automate certain actions. Attention, the onboarding as a Document Consumer is irreversible.

Follow the onboarding self-service process below to integrate as a **Document Consumer** of the e-Box Enterprise. Your enterprise can do some testings in the application *pre-prod* to ensure that everything is OK before using e-Box as a Document Consumer in Production. 

![Diagram DocConsumer Onboarding Process](https://github.com/YiyaoShan/Documentation/blob/main/ProcessusOnboardingDocConsumer.png)


# Onboarding Process to become a Document Consumer 
You’ve not yet used e-Box Enterprise? Before you can become a Document Consumer of e-Box Enterprise, you must create an account and do security settings. 
You already use the e-Box Enterprise? Go directly to the section ***1.2 Accept Terms of Use***.



## Step 1: Onboarding self-service Document Consumer
### 1.1 User account management
In order to become a DocConsumer, you need an **Access Manager (AM)** or a **Co-Access Manager (Co-AM)** to activate and manage your technical "Webservice" channel. Follow the [Procédure pour la gestion des accès dans UMan.pdf](https://www.socialsecurity.be/site_fr/general/helpcentre/rest/documents/pdf/procedure_pour_gestion_des_acces_UMan_FR.pdf)/ [Procedure voor toegangsbeheer in UMan.pdf](https://www.socialsecurity.be/site_nl/general/helpcentre/rest/documents/pdf/procedure_voor_toegangsbeheer_in_uman_NL.pdf) manual to designate an AM/co-AM and manage user access.


| Role EN | Role FR | Role NL |
|------------|-----------|-----------|
| Access Manager (AM) | Gestionnaire d'Accès(GA) = Gestionnaire Local(GL) | Toegangs Beheerder (TB)= Lokale Beheerder (LB) |
| co-Access Manager (co-AM) | co-Gestionnaire d'Accès (co-GA) =co-Gestionnaire Local(co-GL) | co-Toegangs Beheerder (co-TB)=Lokale co-Beheerder (co-LB) |

### 1.2 Accepting the terms of use
In order to use e-Box as a Document Consumer, your enterprise must accept the terms of use for Document Consumer in your [**e-Box**](https://www.eboxenterprise.be/). Go to the section **Document Consumer** in the menu ***Gestion e-Box***/ ***Beheer e-Box***. You'll have to read the terms of use for Document Consumer, ***scroll to the end*** of the complete terms of use and accept the terms of use. Only **after the acceptation** can your enterprise create a technical account, which allows you to consult your messages as a Document Consumer.


### 1.3 Creating a Webservice account in ChaMan
The AM/co-AM should now create a Webservice account in [ChaMan](https://chaman.socialsecurity.be/). To do this, follow the steps below:
1.Connect yourself to your enterprise, then click on *Ajouter un compte Webservice*/*Een Webservice account toevoegen*.
2.Choose 'REST' as type and give a name to your account.
3.Select **DocConsumer e-Box Enterprise**, upload your certificate and give it a name.
The certificate to be used must be a **X.509 certificate**. Any official issuer is acceptable. Please avoid using a self-signed certificate. Make sure you follow the [expected format](https://dev.eboxenterprise.be/docs/common/x509_certificate). 
4.Click on *Valider*/*Bevestigen*.
5.After the validation, you can find your **OAuth client ID** under the section *Webservices Account*.

![Diagram Add ChaMan account](https://github.com/YiyaoShan/Documentation/blob/main/AjoutCOMPTEWS.png)

![Diagram OAuth client ID](https://github.com/YiyaoShan/Documentation/blob/main/CLIENTID.png)

## Step 2: OAuth configuration
### 2.1 enterprise registration in OAuth Server
The previous step involves the registration of your enterprise (identified by its BCE-KBO number) in the Social Security OAuth authorization server, thanks to which you get an **OAuth client ID** that is associated with your enterprise and your certificate, allowing you to generate an **OAuth AccessToken** which serves for consulting the e-Box Enterprise RESTful API. 

### 2.2 Retrieving the OAuth AccessToken
The Social Security’s RESTful Webservice is highly secured. Before you can access to the e-Box Enterprise RESTful Webservice, you must request an OAuth AccessToken from the OAuth authorization server. 
The [OAuth introspect](https://github.com/e-Box-Enterprise-Belgium/examples/tree/master/ouath-introspect) example shows how an OAuth AccessToken can be retrieved. You must use your certificate to authenticate your enterprise. The ``GetAccessTokenV5.getAccessToken()`` method is responsible for obtaining the token.

<table>
<tr><td>OAuth Authorization Server URL</td><td>https://services.socialsecurity.be/REST/oauth/v5/token</td></tr>
<tr><td>Audience (PRD)</td><td>https://services.socialsecurity.be/REST/oauth/v5/token</td></tr>
</table>

As a Document Consumer, your AccessToken will contain the necessary scopes to perform all possible requests concerning your e-Box:
- **Provider Registry** (``scope:documentmanagement:ebox:enterprise:federation-rest:registry``) to get the list of Document Providers.
- **documentconsumer** (``scope:document:management:consult:ws-eboxrestentreprise:documentconsumer``) to get and perform authorized actions on all messages of your e-Box ;


More info :
- [OAuth2 Integration Client Credential.pdf](https://www.socialsecurity.be/site_fr/general/helpcentre/rest/documents/pdf/doc_portal_oauth2_client_credential_FR.pdf)
- Java code to implement OAuth : [ClientCredentialTokenGenerator.java](https://www.socialsecurity.be/site_fr/general/helpcentre/rest/documents/ClientCredentialTokenGenerator.java)



## Step 3: e-Box Enterprise consultation via REST Webservice
### 3.1 Find the Document Providers
Once you have obtained your token, you can find the list of Document Providers by making a GET request to the Provider Registry (don't forget to use the AccessToken retrieved in the previous step):
| Environment| URL Provider Registry                                                                     |
|------------|------------------------------------------------------------------------------------------------|
| Production | ``https://services.socialsecurity.be/REST/ebox/enterprise/federation/v1/messageProviders``     |

In the response, you will find our Document Provider:

| Environment| Endpoint e-Box enterprise                                                           |
|------------|-------------------------------------------------------------------------------------|
| Production | ``https://services.socialsecurity.be/REST/ebox/enterprise/messageRegistry/v2/``      |

### 3.2 Consulting your e-Box Enterprise via REST Webservice
The consultation of your e-Box is done via an HTTP ***GET*** through all the methods that start with ```/ebox``` defined in [MESSAGE REGISTRY SERVICE](https://dev.eboxenterprise.be/docs/spec/specifications)  of e-Box. This specification is available in ``.yaml`` format. Thus, a DocConsumer can not only view his messages, but also the ReferenceData. The methods in the *"publication"* section are only available for the Document Sender and the Document Provider.

Some points of attention:
- Messages will be considered as read when the main content has been retrieved. On the e-Box Enterprise frontend, the visual distinction is the bolding ("unread") or not ("read") of the message title.
- Messages retrieved by Document Consumer remain available in the e-Box until their expiration date. 
- The e-Box Enterprise application (user interface version) is always available (and must be used in case of errors with the use of technical interfaces).


# The RESTful Webservice for e-Box Enterprise
The RESTful Webservice for e-Box Enterprise offers you several functionalities: 
1. **Messages** : You can not only view all messages in your e-Box by simply calling ```/messages```, but also filter messages by expiration date, sender, [partition](https://dev.eboxenterprise.be/docs/federation/partition), etc. As on the user interface, you can also change the visibility and the partition of messages. 
   - *Partition*: Partitions are "sub-boxes" of the e-Box that have their own identity and their own user access management. They allow to 'classify' the messages and thus each user receives only the messages of his partition instead of all the messages received by his enterprise. This improves security in case of confidential messages and helps to organize the received messages. DocConsumers are not limited to partitions as they can view all messages in the e-Box without applying any filter.
   - *Message forAction*: Some messages require an action from the user. (e.g., print a document) As a DocConsumer, once you have completed the action, you can "execute" the message action by making a "*patch*" via e-Box Enterprise REST Webservice.
2. **ReferenceData**: it allows you to retrieve the details of messageTypes, senderOrganizations and senderApplications.
3. **Statistics**: it tells you how many messages are about to expire, how many messages require action on your part and how many messages are registered messages.
4. **Reply**: you can reply to messages (if applicable) and execute the message action when it has been done.  

# Need more info? 
- [What is e-Box Enterprise](https://wwwacc.eboxenterprise.be/fr/index.html) 
- [What is a partition](https://dev.eboxenterprise.be/docs/federation/partition)
- [e-Box Enterprise Services Specifications](https://dev.eboxenterprise.be/docs/spec/specifications)
