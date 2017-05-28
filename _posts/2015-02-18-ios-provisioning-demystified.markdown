---

title: "iOS Provisioning demystified"
date: 2015-02-18 14:56:59 +0100
comments: true
published: false
categories:
---

# Digital identity
The evolution of technology gave birth to the creation of the concept of **digital identity** or **virtual identity**. As the **identity** in reality, it certificates that some informations about a **physical person** are real.

# L'identité numérique
Le développement des technologies de l'information a conduit à la création du concept **d'identité numérique** ou **virtuelle**, qui comme sa consoeur **réelle** nous permet de garantir **l'authenticité** physique d'une personne. De nombreux concepts en découlent comme la signature ou le cryptage d'information et sont largement utilisés sur internet dans le domaine du développement logiciel à grande[^1] ou petite échelles[^2]. Plus récemment, c'est Apple qui se sert de ces notions pour s'assurer de l'intégrité de la phase de développement d'une application pour leur plateformes.

   [^1]: Signature des "commits" sous *git* pour le dépôt Gnu/Linux - [Developers Certificate of Origin](http://kerneltrap.org/files/Jeremy/DCO.txt "Developers Certificate of Origin")
   [^2]: Signature des paquets dans le projet debian - [SecureApt](http://wiki.debian.org/SecureApt "SecureApt")

La naissance de **l'idendité numérique** est intimement liée à l'évolution de la cryptographie dans le domaine de l'informatique depuis les années 70-80 et particulièrement à la naissance des algorithmes de chiffrement asymétrique[^3].

Ces systèmes reposent sur la notion de **clef publique** et de **clef privée**. Ces sur ces deux options que reposent tout le concept **d'identité numérique**. De manière pratique, vous ne vous préoccupez rarement de la manière à suivre pour créer créer ces deux objets. C'est un logiciel qui vous aide à les créer au moyen d'informations que vous lui fournissez.

   [^3]: Historique - [Wikipedia - Cryptographie asymétrique ](http://fr.wikipedia.org/wiki/Cryptographie_asym%C3%A9trique#Historique "Cryptographie asymétrique")

# Principes fondamentaux du cryptage asymétrique

## Clef privée et clef publique
Il faut voir la clef privée comme la partie à protéger absolument des regards indiscrets. Elle contient un mot de passe que seul le propriétaire de la clef connaît. **Elle permet de décoder toute information chiffrer à l'aide la clef publique du propriétaire**.

De manière inverse, **tout message chiffrer avec une clef privée peut être décoder avec une clef publique**. Les clefs forment une paire de **clefs asymétriques**.[^4].


{% AWS_S3_Image signature.png bucket:yageek-static folder:blog_upload/deploy_adhoc_inhouse %}  

### Chiffrement de message

En d'autre terme, il faut voir la **clef publique comme un cadenas dont l'unique clef est la clef privée** comme sur le schéma ci-dessous.

{% AWS_S3_Image users_keys.png bucket:yageek-static folder:blog_upload/deploy_adhoc_inhouse %}  

Pour que les utilisateurs s'écrivent, ils s'échangent leur clefs publiques et signent leur message avec la clef publique de leur correspondant.

{% AWS_S3_Image users_message.png bucket:yageek-static folder:blog_upload/deploy_adhoc_inhouse %}  

Ensuite à chacun d'utiliser sa clef privée pour décoder le message
{% AWS_S3_Image users_dec.png bucket:yageek-static folder:blog_upload/deploy_adhoc_inhouse %}  

  [^4]: [Principes de la cryptographie moderne  ](http://ser-info-02.ec-nantes.fr/users/info3/weblog/c6a6e/Principes_de_la_cryptographie_moderne.html)


### La signature électronique

Pour la signature numérique, on ne chiffre pas les données mais seulement un empreinte (condensat généralement) crée à partir de ces données que l'on transmet avec le message.
On crée le condensat à l'aide d'une fonction de hachage universelle Ce résultat est chiffré avec la **clef privée** de l'utilisateur. Le destinataire du message crée un nouveau condensat à partir des données reçu. Il déchiffre le condensat reçu plus tôt à l'aide la clef publique correspondante et le compare à celui qu'il a calculé pour juger de l'authenticité de l'envoyeur.

{% AWS_S3_Image signature_sceau.png bucket:yageek-static folder:blog_upload/deploy_adhoc_inhouse %}  

{% AWS_S3_Image certificat.png bucket:yageek-static folder:blog_upload/deploy_adhoc_inhouse %}  

### Certificat électronique
Le processus de signautre numérique passe généralement par un **certificat** et l'intervention d'un **tiers de confiance**.

Un certificat contient (entre autres) :

*    Au moins une clef publique à utiliser pour déchiffrer le condensat et valider les signatures.
*    Une information sur la personne physique qui a généré cette clef publique
*    La signature d'un tiers de confiance qui garantit que cette clef publique appartient à une personne physique
*    Une date de validité

{% AWS_S3_Image certif1.png bucket:yageek-static folder:blog_upload/deploy_adhoc_inhouse %}  

Lorsqu'un utilisateur désire communiquer avec une autre personne, il lui suffit de se procurer le certificat du destinataire. Ce certificat contient le nom du destinataire, ainsi que sa clé publique et est signé par l'autorité de certification. Il est donc possible de vérifier la validité du message en appliquant d'une part la fonction de hachage aux informations contenues dans le certificat, en déchiffrant d'autre part la signature de l'autorité de certification avec la clé publique de cette dernière et en comparant ces deux résultats.

{% AWS_S3_Image certif2.png bucket:yageek-static folder:blog_upload/deploy_adhoc_inhouse %}  


# Utilisation des certificats chez Apple
Apple se sert de son système de clefs présents sur ces plateformes de bureau pour valider le développement et la publication de produits. C'est un système de clef privée et clef publique comme décrit ci-dessus.

Apple opère une distinction stricte entre le développement et la distribution d'une application en créant deux identité numériques distinctes pour chacune.

## Entité de développement
Le développement est constitué de trois types de membres. Chacun des membres représente un ***ID Apple***.

1. **Team Agent** : cette personne représente l'authorité qui a souscrit au programme de développement Apple et possède tous les privilèges sur le portail de développement. Le ***Team Agent*** est également un ***Team Admin***
2. **Team Admin** : second administrateur potentiel qui possède tous les droits sur le portail de développement à l'exception d'adhérer à de nouveaux programmes de développement.
3. **Team Member** : représente un membre de l'équipe de développement qui peut tester son code sur un appareil apple.

## Entité de distribution

L'entité de distribution est unique. Elle est la seule habilitée à produire une application binaire en vue de la déployer.

## Signature de code
Lors de la phase de développement sur appareil iOS ou de la phase de production, le code est signé (cf le paragraphe signature de la première partie) afin de s'assurer que seuls les personnes habilitées à développer ou distribuer du code (ou vérifier qu'ils ont payé leur abonnement annuel) peuvent le faire. Ainsi, XCode va automatiquement utiliser l'application *Trousseaux d'accès* pour vérifier les identités des différentes personnes. Vous devez donc déclarer à Apple toutes les informations nécessaires à l'authentification de chacune des personnes impliquées. Cette déclaration se base sur des **Provisioning Profile** (Profils d'approvisionnement)

{% AWS_S3_Image ios_dev_digital_assets.png bucket:yageek-static folder:blog_upload/deploy_adhoc_inhouse %}

### Déclarer les appareils
Chaque appareil apple possède un identifiant unique. Il peut être trouvé dans l'*Organizer* à l'onglet *Devices* sous XCode.

{% AWS_S3_Image organize.jpg bucket:yageek-static folder:blog_upload/deploy_adhoc_inhouse %}

Un des **Team Admin** doit les déclarer sur le portail de développement.

{% AWS_S3_Image Adding_a_Device.png bucket:yageek-static folder:blog_upload/deploy_adhoc_inhouse %}

### Déclarer les développeurs
Les développeurs doivent posséder chacun une paire de clef privée/publique dans leur trousseau. Vous devez ensuite créer un certificat de développeur à partir de votre clef publique si ce n'est pas déjà fait. Pour ce faire vous devez utliser l'application **Trousseaux d'accès** et cliquer sur Trousseaux d'accès -> Assistant de certification -> Demander une certificat à une autorité de certification et transmettre cette demande au **Team Agent** qui créera et déclarera ce certificat sur le portail. *Les certificats sont valables pendant 1 an. Au delà, il faudra penser à les renouveler*.

La première fois que vous souhaiterez développer sur un appareil, vous devrez vous rendre dans l'*Organizer* à la section *Devices* et cliquer sur **Use for Development**.

{% AWS_S3_Image org1.jpg bucket:yageek-static folder:blog_upload/deploy_adhoc_inhouse %}

Après le clique, si vous n'aviez pas de certificat de développement auraparavant, XCode vous proposera de lancer le processus de création pour vous.

### Déclarer les distributeurs
De la même manière que les développeurs, l'entité qui va distribuer l'application se doit d'avoir un certificat de distribution. Vous pouvez utilisez le portail de développement Apple pour en générer un ou passer par XCode.[^5]

   [^5]:[Managing distribution certificate](http://developer.apple.com/library/ios/#documentation/ToolsLanguages/Conceptual/DevPortalGuide/ManaginganiOSDistributionCertificate/ManaginganiOSDistributionCertificate.html#//apple_ref/doc/uid/TP40011159-CH26-SW1 "Managing distribution certificate")

### Déclarer les applications
Chaque application se distingue d'une autre par un identifiant unique. Cet identifiant est choisi à lors de la déclaration de l'application sur le portail par le **Team Agent** est ne peut être modifié par la suite. De manière générale, il prend la forme suivante **com.nomDeLEntreprise.NomDeLApplication**.


### Savoir se présenter auprès d'XCode
Pour vérifier les différentes identités numériques des différents développeur sur les différentes applications sur les différents appareils, Apple a encapsulé les différentes informations liées à l'identité des personnes, des appareils et des applications dans des ***Provisioning Profile***.

#### Pour développer sur un appareil
On se sert alors d'un **Development Provisioning Profile**. Il regroupe trois types d'informations :

1.   Les identifiants des appareils utilisables sur ce profile.
2.   Les certificats des développeurs ayant l'autorisation de développer sur ces appareils.
3.   L'identifiant de l'application autorisée a être testée sur ces appareils.

En d'autres termes, le profil utilisateur stipule qu'un certain nombre de développeurs on le droit de tester une application précise sur un nombre déterminé d'appareil.

Lorsque vous demandez à XCode de lancer votre application sur un appareil, ce dernier va chercher dans votre ordinateur si un **Development Provisioning Profile** correspond à l'identifiant de votre application. Ensuite, il va chercher un certificat de développment et sa **clef privée** dans le trousseau qui correspondent à un certificat de développeur déclaré dans le **Development Provisioning Profile** . Finalement, il regarde si l'appareil sur lequel vous souhaitez développer l'application est déclaré dans le profil.

Si XCode trouve ces trois informations, le code est signé et vous êtes autorisé à lancer l'application sur l'appareil sinon, c'est qu'une des trois vérifications a échoué (identifiant de l'application, certificat de développeur ne correspont pas à une identité dans le trousseau, appareil non déclaré dans le profil)

#### Pour distribuer son application
On se sert alors d'un **Distribution Provisioning Profile**. Il regroupe deux informations :

1. Le certificat du distributeur autorisé à déployer l'application.
2. L'identifiant de l'application autorisée à être déployée.

Lorsque vous demander à XCode de déployer l'application, il va chercher dans le trousseau de l'ordinateur un certificat de distribution et sa **clef privée** correspondants au **certificat de distribution** présent dans le **Distribution Provisioning Profile**. Si la clef n'est pas trouvée, vous ne pourrez pas continer le processus de distribution. L'identifiant de l'application doit également correspondre à celui présent dans le profil **Distribution Provisioning Profile**, sinon elle refusera de s'installer sur les appareil lors du déploiement.

Le **certificat de distribution** se récupère sur le portail des développeurs dans la section Certificates et dans l'onglet Distribution.  

Le **Distribution Provisioning Profile** se récupère dans la section Provisioning puis l'onglet Distribution.

### Préparer son projet à la distribution
On suppose dans cette partie que l'ordinateur utilisé pour la distribution possède :

1. Le **certificat de distribution** et **la clef privée** correspondante
2. **Distribution Provisioning Profile** correspondant à l'application

#### Créer une configuration et un schéma spécifique
Pour ne pas rentrer en conflit entre les signatures liées au développement et les signatures liés à la distribution de créer une configuration spécifique pour le mode de distribution que vous souhaitez (Ad-Hoc, In-House).

Pour ce faire, sous XCode, cliquer sur le projet de votre application et dans l'onglet *Infos* à la section *Configurations*, cliquer sur l'îcone *+* (plus) pour en ajouter une nouvelle.

{% AWS_S3_Image new_conf.png bucket:yageek-static folder:blog_upload/deploy_adhoc_inhouse %}

Nous allons ensuite créer un schéma pour la distribution. Pour ce faire cliquez sur les schéma et choisissez *New Scheme*. Choisissez la cible correspondante et donner un nom au schéma.

{% AWS_S3_Image new_scheme.png bucket:yageek-static folder:blog_upload/deploy_adhoc_inhouse %}

{% AWS_S3_Image new_scheme_2.png bucket:yageek-static folder:blog_upload/deploy_adhoc_inhouse %}

Éditez ensuite le schéma pour choisir la configuration correspondante pour l'archivage

{% AWS_S3_Image conf_scheme_2.png bucket:yageek-static folder:blog_upload/deploy_adhoc_inhouse %}

#### Choix de l'identité pour la signature du code
Il faut maintenant choisir le l'identité utilisée pour signer le code. Pour ce faire aller dans le navigateur de projet et cliquer sur la cible à exporter. Aller dans l'onglet *Build Settings* et choissisez le filtre d'affichage *All*. Tappez *code signing* dans le champ de recherche.

{% AWS_S3_Image code_signing_1.png bucket:yageek-static folder:blog_upload/deploy_adhoc_inhouse %}

{% AWS_S3_Image code_signing_2.png bucket:yageek-static folder:blog_upload/deploy_adhoc_inhouse %}

#### Archivage
Pour déployer l'application, il faut d'abord l'archiver pour la plateforme voulue. Assurez vous d'avoir sélectionner une plateforme iOS, sinon la fonction d'archivage ne sera pas disponible.

{% AWS_S3_Image archiv_1.png bucket:yageek-static folder:blog_upload/deploy_adhoc_inhouse %}

Cliquer ensuite sur le menu *Product* puis *Archive*

{% AWS_S3_Image archiv_2.png bucket:yageek-static folder:blog_upload/deploy_adhoc_inhouse %}

XCode va maintenant compiler votre application en vue de l'exporter. Une fois le processus terminer,vous retrouvez votre archive dans l'*Organizer* dans la partie *Archives*.

{% AWS_S3_Image organize_prod_1.png bucket:yageek-static folder:blog_upload/deploy_adhoc_inhouse %}

### Déploiement
Choisissez la version à déployer dans l'*Organizer* et cliquez sur distribute.

{% AWS_S3_Image dep1.png bucket:yageek-static folder:blog_upload/deploy_adhoc_inhouse %}

Choisissez le mode de déploiement Ad-Hoc ou In-House et cliquez sur *Next*

{% AWS_S3_Image dep2.png bucket:yageek-static folder:blog_upload/deploy_adhoc_inhouse %}

Choisissez l'entité de distribution et cliquez sur *Next*

{% AWS_S3_Image dep3.png bucket:yageek-static folder:blog_upload/deploy_adhoc_inhouse %}

Choissisez l'endroit de sauvegarde de l'application et renseigner les champs correspondant au déploiement.

{% AWS_S3_Image dep4.png bucket:yageek-static folder:blog_upload/deploy_adhoc_inhouse %}
