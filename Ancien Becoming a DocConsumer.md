---
title: Becoming a Document Consumer
---

Attention, the integration as a Document Consumer is currently in pilot phase: Only integrations with the agreement by the NSSO can go live in the production environment. 
Integrations can take place in the acceptance environment in order to create the necessary tools to retrieve messages from the e-Box.
Please note that the user interface offered by the NSSO remains the main and official way of retrieving messages.

Message consultation happens via REST request following the [e-Box RESTful API](../spec/specifications.md).
This specification is available in the ``.yaml`` format.
All the methods excepted these in Publication tag are available for Document Consumers.
So a Document Consumer can consult his messages, reference data and perform replies.
The POST method on messages is available only for [Document Sender](../ds/document_sender.md).

## Getting an Oauth Token for consultation
The [oauth introspect example](https://github.com/e-Box-Enterprise-Belgium/examples/tree/master/ouath-introspect) shows how an Oauth token can be retrieved.
You have to request your AccessToken to the Authorization Server.
The ``GetAccessTokenV4.getAccessToken()`` method is the one responsible of getting the token.
 
<table>
<tr><td>OAuth Authorization Server URL (ACC)</td><td>https://services-acpt.socialsecurity.be/REST/oauth/v5/token</td></tr>
<tr><td>Audience (ACC)</td><td>https://services-acpt.socialsecurity.be/REST/oauth/v5/token</td></tr>
<tr><td>OAuth Authorization Server URL (PRD)</td><td>https://services.socialsecurity.be/REST/oauth/v5/token</td></tr>
<tr><td>Audience (PRD)</td><td>https://services.socialsecurity.be/REST/oauth/v5/token</td></tr>
</table>

Getting a token requires having cleared the OAuth part of the onboarding. If it is not done yet, see the [Document Consumer onboarding process](onboarding_process.md).

You will get the scopes:
- ``scope:document:management:consult:ws-eboxrestentreprise:documentconsumer`` to get and perform actions on all messages in your e-Box;  
- ``scope:documentmanagement:ebox:enterprise:federation-rest:registry`` to get the list of Document Providers

## Endpoints
Once you have got your token, you can find the list of Document Providers to call in the result of a GET request to the *Document Provider Registry*.
You need the scope ``scope:documentmanagement:ebox:enterprise:federation-rest:registry`` in order to consult this web service.
The API is in the [specifications page](../spec/specifications.md#DocumentProviderRegistry)

| Environment| URL Document Provider Registry                                                                     |
|------------|------------------------------------------------------------------------------------------------|
| Acceptance | ``https://services-acpt.socialsecurity.be/REST/ebox/enterprise/federation/documentProviderRegistry/v1/messageRegistries``|
| Production | ``https://services.socialsecurity.be/REST/ebox/enterprise/federation/documentProviderRegistry/v1/messageRegistries``     |

Among them, you can find our Document Provider:

| Environment| Endpoint e-Box enterprise                                                           |
|------------|-------------------------------------------------------------------------------------|
| Acceptance | ``https://services-acpt.socialsecurity.be/REST/ebox/enterprise/messageRegistry/v3/``|
| Production | ``https://services.socialsecurity.be/REST/ebox/enterprise/messageRegistry/v3/``      |

