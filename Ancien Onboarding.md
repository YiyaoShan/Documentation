---
title: Onboarding process to become a Document Consumer
sidebar_label: Onboarding
---

![Diagram: DocConsumer onboarding process](/doc_media/docConsumerOnboardingProcess.png)

## Onboarding form request
To become a new e-Box Enterprise DocConsumer, your enterprise needs to send the following documents to [eBoxIntegration@smals.be](mailto:eBoxIntegration@smals.be).
- The public part of the certificate you will use;
- The completed document “[e-Box DocConsumer request form and terms of use](/doc_media/e-Box_DocConsumer_onboarding_form.docx)”, signed by a Legal Representative of your company.

The certificate needed is a [X.509 certificate](../common/x509_certificate.md).
Any official issuer is acceptable. Please avoid to use a self-signed certificate.
Please pay attention to respect [the expected format](../common/x509_certificate.md).
It’s also important to have a distinct certificate for each work environment (Acceptance, Production).

## OAuth configuration
The e-Box integration team validates your request and configure your enterprise as a new OAuth client.
They will also communicate you the client-ID.
As a DocConsumer, you will get the necessary scopes to perform all the possible requests to manage your e-Box: 
- **summary_own_ebox** (``scope:document:management:consult:ws-eboxrestentreprise:summaryownebox``) to get the summary of your e-Box; 
- **messages_full** (``scope:document:management:consult:ws-eboxrestentreprise:messagesfull``) to get and perform authorized actions on all messages in your ebox;  
- **reference_data** (``scope:document:management:consult:ws-eboxrestentreprise:referencedata``) to retrieve the details of the messageTypes, senderOrganizations, and senderApplications.
- **Provider Registry** (``scope:documentmanagement:ebox:enterprise:federation-rest:registry``) to get the list of Document Providers.

## Technical integration
You can then call the Authorization Server to get your Access token.
With this token, you can [call the different Message Registries](document_consumer.md), in order to retrieve all your e-Box data.
