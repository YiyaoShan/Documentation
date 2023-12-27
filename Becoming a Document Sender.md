---
title: Becoming a Document Sender
---

Publications happen through the ```/publishMessage``` method of the [e-Box RESTful API](../spec/specifications.md).
This method uses a multipart HTTP POST to send up to 6 documents attached to an e-Box Message.
The API fully supports [end to end streaming](#end-to-end-streaming-considerations).

The authentication has to be done via an [OAuth2 token request](#getting-an-oauth-token-for-publication). See the [Document Sender onboarding process](onboarding_process.md) to configure your enterprise as a new OAuth client.

## Avoid duplicate publication
In order to avoid duplicate publication requests, it's recommended to use ***Idempotency-Key*** in the HTTP request header for each publications.

Idempotency-Key is an Item Structured Header [RFC8941](https://www.rfc-editor.org/info/rfc8941). Its value MUST be a String. 
It is RECOMMENDED that UUID [RFC4122](https://www.ietf.org/archive/id/draft-ietf-httpapi-idempotency-key-header-01.html#RFC4122) or a similar random identifier be used as an idempotency key.
Here is an example of the usage of *Idempotency-Key*:
``Idempotency-Key: "8e03978e-40d5-43e8-bc93-6894a57f9324"``

For each publication, the Document Sender SHOULD send a **single and unique** idempotency key in the HTTP Idempotency-Key request header field. The Idempotency-Key used in one request MUST NOT be reused with another request with a different request payload.


## Minimal publication example

The following is pretty much the simplest publication request that can be made. It contains the following HTTP parts:

1) ``messageToPublish``: This part contains the meta information of the message.
Example:
```json
messageToPublish: {
    "recipient": {
      "eboxType": "enterprise",
      "enterpriseNumber": "0123456789"
    },
    "sender": {
      "eboxType": "enterprise",
      "enterpriseNumber": "0123456789"
    },
    "subject": {
      "fr": "Message example title"
    },
    "senderApplicationId": "db2b.SPFETCS-FODWASO.fgov.be",
    "messageTypeId": "SpfFodSocialElections",
    "attachments": [
        {
          "httpPartName": "upfile1",
          "isMainContent": true,
          "isAttachmentSigned": false
        }
    ],
    "isReplyAuthorized": false
}
```

2) ``upfile1``: This part MUST contain the binary and basic meta information of the document. It is a standard HTTP binary file upload part which needs to specify the following information:

- data stream: the raw data of the content
- filename: the file name of the document
  - **Note**: The filename must be encoded if it contains characters outside of the ISO-8859-1 charset (so french accents etc).
  - The javascript "encodeURIComponent" method can be used. So for example "upfilétéstmichaël123 (1) (1).pdf" becomes "upfil%C3%A9t%C3%A9stmicha%C3%ABl123%20(1)%20(1).pdf"
- Content-Type: the [MIME type](https://www.iana.org/assignments/media-types/media-types.xhtml) of the document

```
Content-Disposition: form-data; name="upfile1
"; filename="MyTestDocument.pdf"
Content-Type: application/pdf

(data)
``` 

**Note**: The ``upfile1`` part and the extra meta information of the document are linked together through ``"httpPartName": "upfile1"`` in the ``messageToPublish`` part.

### Response

Provided that the request is correct, one can expect a ``201`` status code to be issued with our without an additional information ``code`` in the JSON response body.

### NO_DIGITAL_USER response code

This code allows the Document Sender to know that the publication was to a recipient that never opened his e-Box.
This is the preferred method for a Sender to determine if the User uses his e-Box or not.
An alternative to this is to use the [e-Box Federation WS](../federation/federation_ws.md) but this requires to integrate with another web service and does not fit the "e-Box First" philosophy we are trying to push. 

### For the attention of

If the publication is for the attention of a particular person, a ``ForTheAttentionOf`` object has to be added in the ``messageToPublish``.
Example:
```json
        "forTheAttentionOf": {
          "ssin": "84022711080",
          "label": "John Doe"
        }
```
The ``type`` is used to determine if the attention is for a person or a group of people. For the moment, only ``"person"`` is suported. The ``id`` property is the National Register Number.

## Minimal publication example for a citizen
If the recipient is a citizen, the example below shows a minimal publication.

```json
messageToPublish: {
    "recipient": {
      "eboxType": "citizen",
      "ssin": "71022722580"
    },
    "sender": {
      "eboxType": "enterprise",
      "enterpriseNumber": "0123456789"
    }
    "subject": {
      "fr": "Message de test"
    },
    "senderApplicationId": "employment:job-attest:werkkaart",
    "messageTypeId": "WK1-ACTIVA",
    "attachments": [
        {
          "httpPartName": "upfile1",
          "isMainContent": true,
          "isAttachmentSigned": false
        }
    ],
    "replyAuthorized": false
}
```

The difference is in the ``recipient`` object. The ``eboxType`` becomes ``citizen`` and the property ``ssin`` replaces ``enterpriseNumber``.
You cannot use a ``messageTypeId`` and a ``senderApplicationId`` that are configured only for enterprise-to-enterprise publications.

Please note that only PDF files work properly for e-Box web application used by citizens.

## Getting an Oauth Token for publication

The [Cookbook OAuth V5](/doc_media/OAUTH-cookbook_V5.docx) introduces OAuth and explains how a token can be retrieved.
You can also see the [oauth introspect example](https://github.com/e-Box-Enterprise-Belgium/examples/tree/master/ouath-introspect) for a java example on how to retrieve an OAuth token.
You have to request your AccessToken to the Authorization Server.
The ``GetAccessTokenV4.getAccessToken()`` method is the one responsible of getting the token.
The scope needed to publish message is ``scope:document:management:consult:ws-eboxrestentreprise:publicationsender``.

<table>
<tr><td>OAuth Authorization Server URL (ACC)</td><td>https://services-acpt.socialsecurity.be/REST/oauth/v5/token</td></tr>
<tr><td>Audience (ACC)</td><td>https://services-acpt.socialsecurity.be/REST/oauth/v5/token</td></tr>
<tr><td>OAuth Authorization Server URL (PRD)</td><td>https://services.socialsecurity.be/REST/oauth/v5/token</td></tr>
<tr><td>Audience (PRD)</td><td>https://services.socialsecurity.be/REST/oauth/v5/token</td></tr>
</table>

Getting a token requires having cleared the OAuth part of the onboarding. If it is not done yet, see the [Document Sender onboarding process](onboarding_process.md).

## Endpoints
Once you have got your token, you can call a method using one of these endpoints:

| Environment| Endpoint e-Box enterprise                                                           |
|------------|-------------------------------------------------------------------------------------|
| Acceptance | ``https://services-acpt.socialsecurity.be/REST/ebox/enterprise/messageRegistry/v3/``|
| Production | ``https://services.socialsecurity.be/REST/ebox/enterprise/messageRegistry/v3``      |

## End to end Streaming Considerations

The order of HTTP parts is arbitrary, each part being linked to its associated meta-data by the ``httpPartName`` property of the publication payload. This allows for end to end streaming on the Document Sender side. See the [Publication Profile Documentation](https://dev.eboxenterprise.be/docs/dp/publication_profile#Order-of-the-HTTP-parts) for more information.

## Examples with Postman
If you use Postman, you might be interested in a [Postman publication examples collection](https://github.com/e-Box-Enterprise-Belgium/examples/tree/master/postman/e-Box%20Enterprise%20REST%20Publication%20examples.postman_collection.json).
After importing that collection,
- Paste your token in the Authorization tab in the menu to edit the collection,
- Replace the ``upfile1`` to ``upfile3`` in the body of the requests with a file in your desktop,
- Replace the sender ``enterpriseNumber``, ``senderApplicationId`` and ``messageTypeId`` with what you are authorized to use,
- Replace the recipient ``enterpriseNumber`` with an existing enterprise. The best is an enterprise that you have an access to consult its e-Box. You can also put the same enterprise number than the sender. That is to say that you can send the message to yourself.

## Our implementation choices

There are some restrictions in our implementation of the service:
- We do not support publication with several languages. Only one among ``fr``, ``nl`` and ``de`` has to be selected in a publication request for the subject, attachment title, body content and business data values.
- We do not support the ``attachmentTitle`` property in the ``AttachmentToPublish`` object. The attachment title will be the file name of the uploaded file.
- Only one main content is accepted. That is to say, either ``isBodyMainContent`` is true or one (and only one) of the ``mainContent`` in attachments is true.
- ``/linkEboxMessage`` feature is not available with the publicationsender scope but broadcast feature still available by asking the procedure to [eBoxIntegration@smals.be](mailto:eBoxIntegration@smals.be).
- We do not support dynamic expiration date. That is to say, in the API about the ``messageToPublish`` object, the ``expirationDate`` property is ignored. The expiration date will be calculated from the current date plus the validity period defined for the message type. You can see the ``validityPeriod`` by doing a GET on ``<endpoint>/refData/messageTypes/<messageType-ID>``
- The business data put in a ``messageToPublish`` can only be those defined for the message type created during the [Onboarding process](onboarding_process.md).
- We do not accept null values. If a property has to be null, like ``"isRegisteredMail": null``, then simply do not put that property.
- We do not support payment data.
- We do not support structuredData.

There is also a maximum length on some strings in a publication request:
- The ``subject`` length is 400 characters maximum.
- The file name of uploaded files are 255 characters maximum.
- The value given to business data are 256 characters maximum.
- The ``name`` in a ``forTheAttentionOf`` object is 100 characters maximum.
