---
title: Onboarding
---

Follow the onboarding self-service process below to integrate as a **Document Consumer** of the e-Box Enterprise. Your enterprise can do some testings with the *pre-prod* URL to ensure that everything is OK before using e-Box as a Document Consumer in Production. Attention, the onboarding as a Document Consumer is irreversible.

![Diagram DocConsumer Onboarding Process](https://github.com/YiyaoShan/Documentation/blob/main/ProcessusOnboarding.png)


# Onboarding Process to become a Document Consumer 
You’ve not yet used e-Box Enterprise? Before you can become a Document Consumer of e-Box Enterprise, you must create an account and do security settings. 
You already use the e-Box Enterprise? Go directly to the section ***1.2 Accepting the terms of use***.



## Step 1: Onboarding self-service Document Consumer
### 1.1 User account management
In order to become a DocConsumer, you need an **Access Manager (AM)** or a **Co-Access Manager (Co-AM)** to activate and manage your technical "Webservice" channel. Follow the [Procédure pour la gestion des accès dans UMan.pdf](https://www.socialsecurity.be/site_fr/general/helpcentre/rest/documents/pdf/procedure_pour_gestion_des_acces_UMan_FR.pdf)/ [Procedure voor toegangsbeheer in UMan.pdf](https://www.socialsecurity.be/site_nl/general/helpcentre/rest/documents/pdf/procedure_voor_toegangsbeheer_in_uman_NL.pdf) manual to designate an AM/co-AM and manage user access.


| Role EN | Role FR | Role NL |
|------------|-----------|-----------|
| Access Manager (AM) | Gestionnaire d'Accès(GA) = Gestionnaire Local(GL) | Toegangs Beheerder (TB)= Lokale Beheerder (LB) |
| co-Access Manager (co-AM) | co-Gestionnaire d'Accès (co-GA) =co-Gestionnaire Local(co-GL) | co-Toegangs Beheerder (co-TB)=Lokale co-Beheerder (co-LB) |

### 1.2 Accepting the terms of use
In order to use e-Box as a Document Consumer, your enterprise must accept the terms of use for Document Consumer in your [**e-Box**](https://www.eboxenterprise.be/). Go to the section **Document Consumer** in the menu ***Gestion e-Box***/ ***Beheer e-Box***. You'll have to read the terms of use for Document Consumer, ***scroll to the end*** of the complete terms of use and accept the terms of use. Only **after the acceptation** can your enterprise create a technical account, which allows you to consult your messages as a Document Consumer.


### 1.3 Creating a Webservice REST account in ChaMan
The AM/co-AM should now create a Webservice REST account in [ChaMan](https://chaman.socialsecurity.be/). To do this, follow the steps below:
1. Connect yourself to your enterprise, then click on *Ajouter un compte Webservice*/*Een Webservice account toevoegen*.
2. Choose 'REST' as type and give a name to your account.
3. Select **DocConsumer e-Box Enterprise**, upload your certificate and give it a name.
The certificate to be used must be a **X.509 certificate**. Any official issuer is acceptable.
4. Click on *Valider*/*Bevestigen*.
5. After the validation, you can find your **OAuth client ID** under the section *Webservices Account*.

![Diagram Add ChaMan account](https://github.com/YiyaoShan/Documentation/blob/main/AjoutCOMPTEWS.png)

![Diagram OAuth client ID](https://github.com/YiyaoShan/Documentation/blob/main/CLIENTID.png)

## Step 2: OAuth configuration
### 2.1 Enterprise registration in OAuth Server
The previous step involves the registration of your enterprise (identified by its BCE-KBO number) in the Social Security OAuth authorization server, thanks to which you get an **OAuth client ID** that is associated with your enterprise and your certificate, allowing you to generate an **OAuth AccessToken** which serves for consulting the e-Box Enterprise RESTful Webservice. 

### 2.2 Retrieving the OAuth AccessToken
The Social Security’s RESTful Webservice is highly secured. Before you can access to the e-Box Enterprise RESTful Webservice, you must request an OAuth AccessToken from the OAuth authorization server. 
The [OAuth introspect](https://github.com/e-Box-Enterprise-Belgium/examples/tree/master/ouath-introspect) example shows how an OAuth AccessToken can be retrieved. You must use your certificate to authenticate your enterprise. The ``GetAccessTokenV5.getAccessToken()`` method is responsible for obtaining the token.

<table>
<tr><td>OAuth Authorization Server URL</td><td>https://services.socialsecurity.be/REST/oauth/v5/token</td></tr>
<tr><td>Audience (PRD)</td><td>https://services.socialsecurity.be/REST/oauth/v5/token</td></tr>
</table>

As a Document Consumer, your AccessToken will contain the necessary scope to perform all possible requests concerning your e-Box:
- **documentconsumer** (``scope:document:management:consult:ws-eboxrestentreprise:documentconsumer``) to get and perform authorized actions on all messages of your e-Box.


More info :
- [OAuth2 Integration Client Credential.pdf](https://www.socialsecurity.be/site_fr/general/helpcentre/rest/documents/pdf/doc_portal_oauth2_client_credential_FR.pdf)
- Java code to implement OAuth : [ClientCredentialTokenGenerator.java](https://www.socialsecurity.be/site_fr/general/helpcentre/rest/documents/ClientCredentialTokenGenerator.java)



## Step 3: e-Box Enterprise consultation via REST Webservice
### 3.1 Testings via the Pre-Prod URL
Once you have obtained your token, for security reasons, you can do some testings on the consultation of your messages via the Pre-Prod URL before consulting your messages in Production as a Document Consumer(don't forget to use the AccessToken retrieved in the previous step): ``https://services-sim.socialsecurity.be/REST/ebox/enterprise/messageRegistry/consultationIntegrationTest/v3``. 

The consultation of messages is done via an HTTP ***GET*** through all the methods that start with ```/ebox``` defined in [MESSAGE REGISTRY SERVICE](https://dev.eboxenterprise.be/docs/spec/specifications)  of e-Box. This specification is available in ``.yaml`` format. Thus, a DocConsumer can not only view his messages, but also the ReferenceData. The methods in the *"publication"* section are only available for the Document Sender and the Document Provider.

### 3.2 Find the Document Providers
If the testings are okay, you can then find the list of Document Providers by making a GET request to the Provider Registry:
| Environment| URL Provider Registry                                                                     |
|------------|------------------------------------------------------------------------------------------------|
| Production | ``https://services.socialsecurity.be/REST/ebox/enterprise/federation/v1/messageProviders``     |

In the response, you will find our Document Provider:

| Environment| Endpoint e-Box enterprise                                                           |
|------------|-------------------------------------------------------------------------------------|
| Production | ``https://services.socialsecurity.be/REST/ebox/enterprise/messageRegistry/v2/``      |

### 3.3 Consulting your e-Box Enterprise via Webservice REST
Now that evernthing is okay, you can consult your e-Box Enterprise via its Webservice REST in Production as a Document Consumer.

Some points of attention:
- Messages will be considered as read when the main content has been retrieved. On the e-Box Enterprise frontend, the visual distinction is the bolding ("unread") or not ("read") of the message title.
- Messages retrieved by Document Consumer remain available in the e-Box until their expiration date. 
- The e-Box Enterprise application (user interface version) is always available (and must be used in case of errors with the use of technical interfaces).

