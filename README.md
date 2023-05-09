---
title: e-Box Enterprise via Webservice
sidebar_label: Onboarding
---

Plus envie de consulter les messages un par un ? Devenez Document Consumer. Vous pouvez consulter tous les messages que vous avez reçus dans l’e-Box Enterprise **en un seul clic** via un **Webservice REST**. Suivez le processus self-service ci-dessous afin de vous intégrer en tant que Document Consumer de l’e-Box Enterprise. L’intégration peut se faire en environnements Acceptance et Production. 

![Diagram DocConsumer Onboarding Process](https://github.com/YiyaoShan/Documentation/blob/main/DocConsumer%20Onboarding%20Processus.png)

# Processus d'intégration pour devenir un Document Consumer (DocConsumer)

Vous n’utilisez pas encore l’e-Box Enterprise ? Avant de pouvoir devenir un DocConsumer de l’e-Box Enterprise, vous devez créer un compte pour vous-même ou pour votre donneur d'ordre et paramétrer la sécurité. 
Vous utilisez déjà l’e-Box Enterprise ? Voir directement section ***«1.2 Accepter les conditions d’utilisation»***

## Etape1: Onboarding self-service DocConsumer
### 1.1 Gestion des comptes utilisateurs
Afin de devenir un DocConsumer, il vous faut un **Gestionnaire d’Accès  (GA)** ou un **co-Gestionnaire d’Accès (co-GA)** pour activer et gérer votre canal technique « Webservice ». Suivez le manuel [Procédure pour la gestion des accès dans UMan.pdf](https://www.socialsecurity.be/site_fr/general/helpcentre/rest/documents/pdf/procedure_pour_gestion_des_acces_UMan_FR.pdf) pour désigner un GA/co-GA et gérer les accès des utilisateurs.
### 1.2 Accepter les conditions d’utilisation
Actuellement, l'activation des canaux WebServices se fait en mode "tout ou rien" : votre entreprise a l’accès au non seulement Webservice e-Box Enterprise, mais aussi à [d’autres services dans le cadre de la Sécurité Sociale](https://www.socialsecurity.be/site_fr/employer/infos/online-services.htm.) . En outre, l’accès se fait toujours au nom de votre entreprise. La responsabilité reste sur l'entreprise en cas de mauvaise utilisation du service / des credentials. Pour la sécurité des connexions, votre entreprise est obligée d’accepter les conditions d’utilisations de DocConsumer.
### 1.3 Créer un utilisateur technique
Le GA/co-GA doit maintenant créer un utilisateur technique dans le service « [Gestion des accès](https://www.socialsecurity.be/site_fr/employer/applics/umoe/index.htm) ». Pour ce faire, consultez le manuel [Créer votre canal Webservice sur le portail de la Sécurité Sociale.pdf](https://www.socialsecurity.be/site_fr/general/helpcentre/rest/documents/pdf/webservices_creer_le_canal_FR.pdf).
Le certificat à utiliser doit être un **certificat X.509**. Tout émetteur officiel est acceptable. Veuillez éviter d'utiliser un certificat auto-signé. Veillez à respecter le [format attendu](https://dev.eboxenterprise.be/docs/common/x509_certificate). Il est également important d'avoir **un certificat distinct pour chaque environnement** (acceptation, production).
*P.S. Gestionnaire Local (GL)=Gestionnaire d’accès (GA) ; co-Gestionnaire Local (co-GL)=co-Gestionnaire d’accès (co-GA)*
### 1.4 Activer un canal Webservice
Une fois l'utilisateur technique créé, votre GA/co-GA doit activer le canal « Webservice » dans le service « [Gestion des accès](https://www.socialsecurity.be/site_fr/employer/applics/umoe/index.htm) ». Référez-vous au manuel [Ajouter votre canal Webservice sur le portail de la Sécurité Sociale.pdf](https://www.socialsecurity.be/site_fr/general/helpcentre/rest/documents/pdf/webservices_ajouter_le_canal_FR.pdf) pour activer le canal.


## Etape2: Configuration OAuth
### 2.1 Enregistrement entreprise dans OAuth Server
L’étape précedente implique l’enregistrement de votre entreprise (identifiée par son n°BCE-KBO) dans le serveur d’autorisation OAuth de la Sécurité Sociale , grâce à laquelle vous obtenez un **client-id OAuth** qui est associé à votre entreprise et votre certificat, permettant de générer un **AccessToken OAuth** pour pouvoir consulter l’API REST de l’e-Box Enterprise. 
### 2.2 Récupération AccessToken OAuth
Le Webservice REST de la Sécurité Sociale  est hautement sécurisé. Avant de pouvoir consulter le Webservice REST de l’e-Box Enterprise, vous devez demander un OAuth AccessToken au serveur d'autorisation OAuth. 
L'exemple [OAuth introspect](https://github.com/e-Box-Enterprise-Belgium/examples/tree/master/ouath-introspect) montre comment un AccessToken OAuth peut être récupéré. Vous devez utiliser votre certificat pour s’authentifier. La méthode ``GetAccessTokenV5.getAccessToken()`` est responsable de l'obtention du token.
<table>
<tr><td>OAuth Authorization Server URL (ACC)</td><td>https://services-acpt.socialsecurity.be/REST/oauth/v5/token</td></tr>
<tr><td>Audience (ACC)</td><td>https://services-acpt.socialsecurity.be/REST/oauth/v5/token</td></tr>
<tr><td>OAuth Authorization Server URL (PRD)</td><td>https://services.socialsecurity.be/REST/oauth/v5/token</td></tr>
<tr><td>Audience (PRD)</td><td>https://services.socialsecurity.be/REST/oauth/v5/token</td></tr>
</table>

En tant que Document Consumer, votre AccessToken contiendra les scopes nécessaires pour effectuer toutes les requêtes possibles afin de gérer votre e-Box :
- **summary_own_ebox** (``scope:document:management:consult:ws-eboxrestentreprise:summaryownebox``) pour obtenir le résumé de votre e-Box ;
- **documentconsumer** (``scope:document:management:consult:ws-eboxrestentreprise:documentconsumer``) pour obtenir et effectuer des actions autorisées sur tous les messages de votre e-Box ;
- **reference_data** (``scope:document:management:consult:ws-eboxrestentreprise:referencedata``) pour récupérer les détails des messageTypes, senderOrganizations et senderApplications.
- **Provider Registry** (``scope:documentmanagement:ebox:enterprise:federation-rest:registry``) pour obtenir la liste des [Documents Providers](https://dev.eboxenterprise.be/docs/dp/document_provider).

Plus d’info :
-	[OAuth2 Integration Client Credential.pdf](https://www.socialsecurity.be/site_fr/general/helpcentre/rest/documents/pdf/doc_portal_oauth2_client_credential_FR.pdf)
-	Code Java pour implémenter OAuth : [ClientCredentialTokenGenerator.java](https://www.socialsecurity.be/site_fr/general/helpcentre/rest/documents/ClientCredentialTokenGenerator.java)

## Etape3: Consultation e-Box Enterprise via Webservice REST
### 3.1 Trouver les Document Providers
Une fois que vous avez obtenu votre token, vous pouvez trouver la liste des Document Providers en faisant une requête GET au Provider Registry (n’oubliez pas d’ utiliser l’AccessToken récupéré à l’étape précédente) :
| Environment| URL Provider Registry                                                                     |
|------------|------------------------------------------------------------------------------------------------|
| Acceptance | ``https://services-acpt.socialsecurity.be/REST/ebox/enterprise/federation/v1/messageProviders``|
| Production | ``https://services.socialsecurity.be/REST/ebox/enterprise/federation/v1/messageProviders``     |

Parmi eux, vous trouverez notre Document Provider :

| Environment| Endpoint e-Box enterprise                                                           |
|------------|-------------------------------------------------------------------------------------|
| Acceptance | ``https://services-acpt.socialsecurity.be/REST/ebox/enterprise/messageRegistry/v2/``|
| Production | ``https://services.socialsecurity.be/REST/ebox/enterprise/messageRegistry/v2/``      |

### 3.2 Consulter votre e-Box Enterprise via Webservice REST
La consultation de votre e-Box se fait via un HTTP GET à travers toutes les méthodes qui commencent par ```/ebox``` définies dans [MESSAGE REGISTRY SERVICE](https://dev.eboxenterprise.be/docs/spec/specifications) de l'e-Box. Cette spécification est disponible au format ``.yaml``. Ainsi, un DocConsumer peut non seulement consulter ses messages, mais aussi les ReferenceData. Les méthodes dans la section *« publication »* n'est disponible que pour le Document Sender et le Document Provider.

Quelques points d’attention :
- Les messages seront considéré comme lu lorsque le contenu principal a été récupéré. Sur le frontend e-Box Enterprise, la distinction visuelle est la mise en gras ("non-lu") ou non ("lu") du titre du message.
- Les messages récupérés par DocConsumer restent disponibles dans l'e-Box jusqu'à leur date d'expiration. 
- L'application e-Box Enterprise (version interface utilisateur) reste toujours disponible pour consulter l'e-Box (et doit être utilisée en cas d'erreur avec l'utilisation des interfaces techniques).

# Le Webservice REST pour e-Box Enterprise
Le Webservice REST pour e-Box Enterprise vous offre plusieurs fonctionnalités.
1. **Messages** : vous pouvez non seulement consulter tous les messages dans votre e-Box en appelant simplement /messages, mais aussi filtrer les messages en fonction de la date d’expiration, de l’expéditeur, de la [partition](https://dev.eboxenterprise.be/docs/federation/partition), etc. Comme sur l’interface utilisateur, vous pouvez également changer la visabilité et la partition des messages. 
   - *Partition* : Les partitions sont des « sous-boîtes » de l’e-Box qui ont leur propre identité et leur propre gestion de l'accès des utilisateurs. Elles permettent de 'classifier' les messages et ainsi chaque utilisateur reçoit juste les messages de sa partition au lieu de tous les messages reçu par son entreprise. Cela améliore la sécurité en cas des messages confidentiels et aide à organiser les messages reçus. Les DocConsumers ne sont pas limités aux partitions vu qu’ils peuvent consulter tous les messages de l’e-Box sans appliquer de filtre.
   - *Message forAction* : Certains messages requièrent une action de la part de l’utilisateur. (e.g., imprimer un document) Une fois que vous aurez terminé l’action, vous pourrez « exécuter » le message action en faisant un « *patch* » via Webservice REST e-Box Enterprise.
2. **ReferenceData** : cela vous permet de récupérer les détails des messageTypes, senderOrganizations et senderApplications.
3. **Statistics** : grâce à cela vous saurez combien de messages vont bientôt expirer, combien de messages requièrent une action de votre part et combien de messages sont des recommandés.
4. **Reply** : vous pouvez répondre aux messages (si applicable) et exécuter l’action du message quand cela a été fait.  

# Besoin de plus d’info ? 
- [Qu’est-ce que l’e-Box](https://wwwacc.eboxenterprise.be/fr/index.html)
- [Qu’est-ce qu’une partition](https://dev.eboxenterprise.be/docs/federation/partition)
- [e-Box Enterprise Services Specifications](https://dev.eboxenterprise.be/docs/spec/specifications)


