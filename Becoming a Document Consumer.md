---
title: Becoming a Document Consumer
---

Does your enterprise receive a large number of documents in your e-Box? Is it difficult for you to process messages manually in your e-Box online? Integration with e-Box via a technical interface (RESTful Webservice) can be a solution, enabling messages to be retrieved in "packages" ("Document Consumer" solution).
You can then get directly all your e-Box messages from your internal IT system. This enables you to manage message distribution within your enterprise. You can also automate certain actions. Attention, the onboarding as a Document Consumer is irreversible.

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
