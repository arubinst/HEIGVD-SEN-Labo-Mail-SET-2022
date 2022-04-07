# Teaching-HEIGVD-SEN-2022-Laboratoire-Docker-Mail et SET


## Introduction

L'un des outils le plus important dans l'arsenal d'un ingénieur social c'est l'email.

L'ingénierie sociale implique très souvent des interactions avec la cible. Ces interactions sont facilitées de nos jours par l'ubiquité des communications numériques. Il est tout à fait normal de recevoir des instructions par email de la part de supérieurs hiérarchiques ou des collègues pour réaliser toute sorte de tâches. Les destins de pays, des opérations financières multimilliardaires, des plans de construction d'un nouvel appareil; tout peut être transmis par email.

L'email est aussi utilisé pour les relations privées. Ça peut être donc la voie de communication avec des membres de la famille, des amis, les conjoints, les fils et parents.

Pour complémenter le travail réalisé pour apprendre à connaitre la cible afin de la compromettre, l'ingénieur social a souvent besoin de délivrer une payload utilisant le même canal de communication. L'utilisation d'un serveur mail public devient parfois compliqué. Si on essaie de faire une campagne de phishing, par exemple, l'émission d'un grand nombre de messages à des destinations différentes peut soulever une alarme qui vous met dans une liste noire. L'utilisation de certaines payloads peut aussi être détectée par votre serveur mail d'envoi.

Il est donc très intéressant de ne pas dépendre d'un serveur mail public. Vous pouvez configurer votre propre serveur email avec quelques manipulations très simples. Ce serveur vous appartient. Il acceptera de faire tout ce que vous lui demanderez de faire, sans poser des questions.


## Une petite note sur l'éthique

Il n'est absolument pas acceptable d'attaquer quelqu'un pour quelque raison que ce soit.

L'utilisation de ces outils à des fins autres que votre propre éducation et formation sans autorisation est strictement interdite par les politiques de ce cours et de l'école, ainsi que par les lois.

Le but de cet exercice est de vous permettre de vous familiariser avec les outils et comment ils peuvent être utilisés dans le contexte professionnel d'un pentest. Ça vous permettra aussi de comprendre les tactiques de l'adversaire afin de pouvoir les contrer par le biais de la politique, de l'éducation et de la formation.


## Que faut-il faire ?

Voici les activités à réalise dans ce laboratoire. Vous devez :

- Installer, configurer et tester votre propre serveur mail
- Installer le Social Engineering Toolkit (SET)
- Créer un collecteur d'identifiants (credential harvester)
- Capturer certains identifiants utilisateur (les vôtres)
- Créer une attaque de mailing utilisant SET et votre propre serveur mail

Le "rapport" de ce labo est très simple : **Pour chaque tâche, faites des captures d'écran de vos activités et répondez les éventuelles questions**.

## Docker Mailserver

Le projet [Docker Mailserver](https://github.com/docker-mailserver/docker-mailserver) est un système très complet et sophistiqué qui vous permet d'installer et utiliser votre propre serveur mail. Il est en même temps très simple et inclut des fonctionnalités avancées comme des filtres de spam et antivirus (justement ce que l'on veut éviter...).

Vous pouvez visiter le site du projet et apprendre beaucoup de choses à propos de cet outil très puissant. Nous allons pourtant nous contenter de faire un nombre assez réduit de manipulations dans le but de le faire fonctionner rapidement et avec peu d'effort. Ce n'est de loin la bonne utilisation de ce produit. En effet, il peut être déployé pour une utilisation en production.

Notre scénario c'est celui d'un attaquant qui se sert de ce serveur pour délivrer des emails de phishing, par exemple, voir des payloads. Notre configuration de base ne nous permettra pas de recevoir des réponses aux mails. Si vous voulez être capable de recevoir des réponses (ce qui peut être le cas dans certains scénarios), il faudra faire du travail supplémentaire pour installer et configurer un serveur DNS capable de fournir des "MX records". La [documentation très complète de Docker Mailserver](https://docker-mailserver.github.io/docker-mailserver/edge/) contient entre autres les informations nécessaires pour configurer votre DNS. L'installation de certificats est aussi normalement importante, mais ce ne sera pas fait pour le moment.

### Configuration minimaliste de Docker Mailserver

Nous avons testé ce guide sur Kali Linux et macOS Monterey. Son utilisation devrait être possible sur Windows avec peu ou pas de modification. Il vous faudra comprendre votre propre infrastructure afin de faire interagir correctement tous les éléments.

Nous allons commencer par créer un répertoire "mailserver" et entrer dedans (ce guide part du principe que vous avez déjà Docker et Docker Compose installés et correctement configurés sur votre plateforme).

```bash
mkdir mailserver
cd mailserver
```
Ensuite, nous allons télécharger les 3 fichiers indispensables pour déployer le serveur. Vous n'êtes pas obligés de cloner le repo github entier.

```bash
DMS_GITHUB_URL='https://raw.githubusercontent.com/docker-mailserver/docker-mailserver/master'
wget "${DMS_GITHUB_URL}/docker-compose.yml"
wget "${DMS_GITHUB_URL}/mailserver.env"
wget "${DMS_GITHUB_URL}/setup.sh"
```

Finalement, nous allons rendre executable le script ```setup.sh```.

```bash
chmod a+x ./setup.sh
```

Le fichier ```mailserver.env```contient une énorme quantité de [variables d'environnement](https://docker-mailserver.github.io/docker-mailserver/edge/config/environment/) qui vous permettent de configurer votre serveur. La bonne nouvelle c'est que la configuration de base est déjà une version "clé en main". Vous pourriez ne rien modifier. Nous allons pourtant éditer le fichier et faire deux petits changements.

Ouvrez le fichier ```mailserver.env``` avec votre éditeur de texte préféré et trouvez la ligne qui fait référence à Amavis. Changez la ligne pour désactiver son utilisation:

```bash
ENABLE_AMAVIS = 0
```

---
#### Question : quelle est l'utilité de cette option ? C'est quoi Amavis ?

```
Réponse : Amavis est un filtre open-source pour le courrier électronique. Ce filtre permet la détection de spams, de virus, de contenus et messages interdits, de rediriger le courrier en fonction de son contenu, de mettre en quarantaine ou archiver les messages, etc.... Dans ce labo, cette option sera désactivée et Amavis ne sera donc pas activé.
```

Cherchez ensuite la variable ```PERMIT_DOCKER``` dans ce même fichier et dans la documentation. Changez sa valeur à :

```bash
PERMIT_DOCKER=connected-networks
```

#### Question : Quelles sont les différentes options pour cette variable ? Quelle est son utilité ? (gardez cette information en tête si jamais vous avez des problèmes pour interagir avec votre serveur...)

```
Réponse : Cette option permet de définir différentes options pour l'option mynetworks. C'est-à-dire que l'on peut choisir depuis quels réseaux il est possible d'envoyer des mails.
Les différentes options sont:
none => Force explicitement l'authentification 
container => Adresse IP du conteneur uniquement 
host => Ajoute un réseau de conteneur Docker (ipv4 uniquement) 
network => Ajoute tous les réseaux de conteneurs docker (ipv4 uniquement) 
connected-networks => Ajoute tous les réseaux docker connectés (ipv4 uniquement)
```
---

Vous allez maintenant éditer le fichier ```docker-compose.yml```. Ce fichier contient aussi une configuration de base qui est fonctionnelle sans modification. Vous pouvez pourtant changer le ```domainname``` dans ce fichier. Vous pouvez choisir ce qui vous convient. Vous voulez utiliser ```gmail.com```? Allez-y ! C'est votre serveur !

La dernière partie de la configuration c'est la création d'un compte que vous pouvez utiliser pour envoyer vos emails. Il suffit d'utiliser la commande suivante, avec évidement les paramètres que vous désirez. Ce compte sera utilisé pour vous authentifier auprès de votre serveur mail :

```bash
./setup.sh email add vladimir@putin.ru password
```

Où ```vladimir@putin.ru```  c'est l'adresse email et le nom d'utilisateur qui seront crées et ```password``` est le mot de passe correspondant.

### Installation et test

C'est le moment de télécharger l'image, créer le container et tester votre serveur. On utilise docker-compose :

```bash
docker-compose -f docker-compose.yml up -d
```

Vous pouvez vous servir de la commande ```docker ps``` pour vérifier que votre container est créé et en fonctionnement.

Nous allons faire un test très basique pour nous assurer que le serveur fonctionne. Vous aurez besoin de ```telnet``` ou d'une commande équivalente (vous pouvez utiliser netcat, par exemple):

```bash
telnet localhost 25
```

Si votre serveur fonctionne correctement, il devrait vous saluer avec :

```bash
Connection to localhost port 25 [tcp/smtp] succeeded!
220 mail.whitehouse.gov ESMTP
```

Dans mon cas, j'ai configuré le domaine de mon serveur avec ```whitehouse.gov```

Vous pouvez ensuite établir une conversation avec votre serveur. Nous allons en particulier nous authentifier. Si vous ne vous authentifiez pas, le serveur refusera de vous laisser l'utiliser comme un relay (Relay access denied).

La commande pour l'authentification c'est ```AUTH LOGIN```. Vous devez ensuite transmettre votre username et votre mot de passe de l'utilisateur que vous avez créé, tous les deux en base64.

Voici ma conversation avec mon serveur :

```bash
arubinst@mailserver % nc -v localhost 25
Connection to localhost port 25 [tcp/smtp] succeeded!
220 mail.whitehouse.gov ESMTP
HELO rubinstein.gov
250 mail.whitehouse.gov
AUTH LOGIN
334 VXNlcm5hbWU6
dmxhZGltaXJAcHV0aW4ucnU=    <----- "vladimir@putin.ru" en base64
334 UGFzc3dvcmQ6
cGFzc3dvcmQ=                <----- "password" en base64
235 2.7.0 Authentication successful
```

---

#### Faire une capture de votre authentification auprès de votre serveur mail

```
Livrable : capture de votre conversation/authentification avec le serveur
```

![image-20220402135528661](figures/image-20220402135528661.png)

---

### Configuration de votre client mail

Cette partie dépend de votre OS et votre client mail. Vous devez configurer sur votre client les paramètres de votre serveur SMTP pour pouvoir l'utiliser pour envoyer des messages.

---

### Montrez-nous votre configuration à l'aide d'une capture

```
Livrable : capture de votre configuration du serveur SMTP sur un client mail de votre choix
```

![image-20220402152149258](figures/image-20220402152149258.png)

---

Vous pouvez maintenant vous servir de votre serveur SMTP pour envoyer des mails. Envoyez-vous un email à votre adresse de l'école pour le tester.

Si tout fonctionne correctement, envoyez-nous (Stéphane et moi) un email utilisant votre serveur. Puisque vous avez certainement créé un faux compte email, n'oubliez pas de signer le message avec votre vraie nom pour nous permettre de vous identifier.

---
```
Livrable : capture de votre mail envoyé (si jamais il se fait bloquer par nos filtres de spam...
```
![image-20220403115345163](figures/image-20220403115345163.png)

---

## The Social-Engineer Toolkit (SET)

### A propos de SET

Selon la propre description donnée par [TrustedSec, LLC](https://www.trustedsec.com), la société de consulting américaine responsable du développement de ce produit, le [Social-Engineer Toolkit](https://github.com/trustedsec/social-engineer-toolkit/) est un framework de test d'intrusion open-source conçu pour l'ingénierie sociale. Le SET dispose d'un certain nombre de vecteurs d'attaque personnalisés qui vous permettent de réaliser rapidement une attaque crédible.

Le SET est spécifiquement conçu pour réaliser des attaques avancées contre l'élément humain. Il est rapidement devenu un outil standard dans l'arsenal des testeurs de pénétration. Les attaques intégrées dans la boîte à outils sont conçues pour être des attaques ciblées contre une personne ou une organisation utilisées lors d'un test de pénétration.

La réalité c'est que, en raison de l'évolution très rapide en matière de protection, cet outil fonctionne que partiellement. C'est un peu le jeu du chat et la souris. Le support pour certaines fonctionnalités est souvent utilisable pendant un certain temps et puis, rendu inutile. Cela reste quand-même très intéressant à le surveiller et à l'essayer.


### Téléchargement et installation de SET

Le SET est nativement supporté sur Linux et sur Mac OS X (experimental). Il est normalement préinstallé sur Kali Linux et il est capable de se mettre à jour lui-même.

Pour une installation sur Ubuntu/Debian/Mac OS X (ou si vous ne le retrouvez pas sur Kali) :

```
git clone https://github.com/trustedsec/social-engineer-toolkit/ setoolkit/
cd setoolkit
pip3 install -r requirements.txt
python setup.py
```
### Execution de SET

Pour exécuter SET, dans votre terminal taper :

```
setoolkit
```

Dépendant de votre OS et de votre installation particulière, il est possible que certaines fonctionnalités ne soient pas disponibles au moins d'utiliser ```sudo```.

```
sudo setoolkit
```

### Credential Harvesting

Vous découvrirez l'un des outils les plus couramment utilisés par les ingénieurs sociaux et les acteurs malveillants pour tromper les cibles.

Nous allons essayer avec le site de Postfinance.

Dans le menu de SET, sélectionner l'option 1, attaques de Social Engineering.

![Menu principal SET](images/harvester1.png)

Ensuite, l'option 2 vous permettra de sélectionner les attaques Web.

![Menu principal SET](images/harvester2.png)

Vous voulez maintenant l'option 3 pour le collecteur d'identifiants.

![Menu principal SET](images/harvester3.png)

Et pour finir, l'option 2 pour cloner un site web.

![Menu principal SET](images/harvester4.png)

Il faudra maintenant remplir deux informations :

(1) l'adresse IP qui réceptionne la requête POST de votre site cloné. Dans notre cas, vous allez très probablement laisser la valeur par défaut proposée par SET (votre adresse dans le NAT d'une machine virtuelle ou votre adresse locale). Si votre attaque est sur une cible externe et que vous récoltez les identifiants depuis un réseau local derrière un NAT, il vous faudra votre adresse publique et faire quelques manipulations de redirection de ports au niveau de votre routeur.

(2) L'url du site à cloner.

Certains sites ne fonctionnent pas bien, voir pas du tout. Pour ces cas, il existe la possibilité de modifier localement le clone du site pour le faire fonctionner. On ne va pas le faire dans le cadre de ce labo.

On a pourtant trouvé deux sites qui fonctionnent bien et que vous pouvez essayer. On avait déjà mentionné Postfinance. L'autre site, c'est notre cher et vénérable gaps :

- ```https://www.postfinance.ch/ap/ba/ob/html/finance/home?login```
- ```https://gaps.heig-vd.ch/consultation/```

---

#### Soumettre des captures d'écran

Pour le collecteur d'identifiants, montrez que vous avez cloné les deux sites proposés. Dans chaque cas, saisissez des fausses informations d'identification sur votre clone local, puis cliquez le bouton de connexion. Essayez d'autres sites qui puissent vous intéresser (rappel : ça ne marche pas toujours). Faites des captures d'écran des mots de passe collectés dans vos tests avec SET.

#### PostFinance

<img src="figures/image-20220402213332566.png" alt="image-20220402213332566" style="zoom:67%;" />

![image-20220407112253185](figures/image-20220407112253185.png)



#### GAPS

![image-20220402213751468](figures/image-20220402213751468.png)

Nous remarquons quelques problèmes d'affichage (le Accès école devenu Accs cole).

![image-20220407112440013](figures/image-20220407112440013.png)

#### Instagram

J'ai voulu tester https://www.instagram.com/?hl=fr mais cela n'a pas fonctionné. J'ai bien eu l'écran de chargement d'Instagram mais la page est ensuite devenue blanche.

#### RTS

<img src="figures/image-20220403104313166.png" alt="image-20220403104313166" style="zoom:67%;" />

![image-20220407112704358](figures/image-20220407112704358.png)

---

### Mass Mailer Attack

Essayez la fonction d'envoie de mails. Vous la trouvez dans "Social Engineering Attacks".

Sélectionnez l'option "Single Email Address". Vous avez le choix entre des modèles de mail préfabriqués ou de créer votre propre message.

Pour cet exercice, nous allons utiliser notre serveur mail que vous venez de configurer.

Les paramètres à remplir sont :

- Adresse email de destination (cible) - vous pouvez essayer votre adresse email de l'école, par exemple
- Sélectionner l'option "User your own server"
- From address : l'adresse email de l'expéditeur de votre message - à vous de choisir le personnage
- FROM NAME : le nom qui sera affiché dans le client mail de la cible
- Username open-relay : le compte que vous avez créé pour votre serveur mail
- Password open-relay : le mot-de-passe que vous avez donné à ce compte
- SMTP server : normalement ce sera ```localhost``` mais ça peut dépendre de votre cas
- Port : 25
- Flag high priority : à vous de choisir
- Joindre une pièce : pas en ce moment. Il faut répondre "n" deux foix

En fonction de beaucoup de paramètres (config de votre serveur mail, par exemple), il est fort probable que votre mail se fasse arrêter par le filtre de spam. Vous pouvez regarder [le filtre de spam de l'école](https://quarantine.heig-vd.ch). Si vous retrouvez votre mail, utilisez l'option "Deliver" pour le libérer. Vous retrouverez votre mail dans la boîte de réception.

Si votre mail s'est fait filtrer, lire les entêtes et analyser les informations rajoutées par le filtre de spam.

---
#### Question : Est-ce que votre mail s'est fait filtrer ? qu'es-ce qui a induit ce filtrage ?

```
Réponse : Oui, voici les informations rajoutées par le filtre de spam:
X-Barracuda-Spam-Report: Code version 3.2, rules version 3.2.3.97098
	Rule breakdown below
	 pts rule name              description
	---- ---------------------- --------------------------------------------------
	0.14 MISSING_MID            Missing Message-Id: header
	0.01 FROM_EXCESS_BASE64     From: base64 encoded unnecessarily
	1.40 MISSING_DATE           Missing Date: header
	1.05 FROM_EXCESS_BASE64_2   From: base64 encoded unnecessarily
	0.10 RDNS_DYNAMIC           Delivered to trusted network by host with
	                           dynamic-looking rDNS
	0.50 BSF_SC5_MJ1963         Custom Rule MJ1963
```

![image-20220403132927822](figures/image-20220403132927822.png)

![image-20220403133139464](figures/image-20220403133139464.png)

Si vous avez une autre adresse email (adresse privée, par exemple), vous pouvez l'utiliser comme cible, soumettre une capture et répondre à la question.

---
#### Question : Est-ce que votre mail s'est fait filtrer dans ce cas-ci ? Montrez une capture.

```
Réponse et capture : Oui, il s'est retrouvé dans mes spams
```

![image-20220403123847741](figures/image-20220403123847741.png)


---

### Explorer les liens "Phishy" et le courrier électronique "Phishy"

Pour cette dernière partie de notre exploration du phishing, nous allons utiliser un contenu réalisé par les  Dr. Matthew L. Hale, le Dr. Robin Gandhi et la Dr. Briana B. Morrison de [Nebraska GenCyber](
http://www.nebraskagencyber.com).

Visitez : [https://mlhale.github.io/nebraska-gencyber-modules/phishing/README/ ](https://mlhale.github.io/nebraska-gencyber-modules/phishing/README/) et passez en revue les modules :

- Analyse d'url. **Ce module risque d'être beaucoup trop simple pour vous** mais il peut être très intéressant pour vos rapports de pentest, surtout comme outil pour sensibiliser les employés d'une entreprise. Gardez-le précieusement comme une partie de votre toolbox pour l'avenir.
- Analyse d'Email (ce module est probablement plus intéressant techniquement pour vous)

En général, c'est un bon exemple de matériel de formation et d'éducation qui peut aider à lutter contre les attaques de phishing et à sensibiliser le personnel d'une organisation.

Vous avez la liberté de reproduire et d'utiliser ce matériel grâce à sa licence.


#### Soumettre des captures d'écran

Pour cette tâche, prenez des captures d'écran de :

- Vos inspections d'un en-tête de courrier électronique à partir de votre propre boîte de réception

Courrier électronique que j'ai envoyé avec SET sur mon adresse mail de la HEIG-VD (avec modifications des adresses IP) qui a été considéré comme SPAM:

```
Received: from EIMAIL02.einet.ad.eivd.ch (10.192.41.72) by
 EIMAIL01.einet.ad.eivd.ch (10.192.41.71) with Microsoft SMTP Server
 (version=TLS1_2, cipher=TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256) id
 15.1.2375.24 via Mailbox Transport; Sun, 3 Apr 2022 13:31:03 +0200
Received: from EIMAIL01.einet.ad.eivd.ch (10.192.41.71) by
 EIMAIL02.einet.ad.eivd.ch (10.192.41.72) with Microsoft SMTP Server
 (version=TLS1_2, cipher=TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256) id
 15.1.2375.24; Sun, 3 Apr 2022 13:31:03 +0200
Received: from mail01.heig-vd.ch (10.192.222.28) by EIMAIL01.einet.ad.eivd.ch
 (10.192.41.71) with Microsoft SMTP Server (version=TLS1_2,
 cipher=TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256) id 15.1.2375.24 via Frontend
 Transport; Sun, 3 Apr 2022 13:31:03 +0200
X-ASG-Debug-ID: 1648980499-1114bd34067cbef0001-vcqECt
Received: from mail.example.com (00-00-000-000.dclient.hispeed.ch [00.00.000.000]) by mail01.heig-vd.ch with ESMTP id 9wY1qyRsakn2KQk7 for <alexandra.cerottini@heig-vd.ch>; Sun, 03 Apr 2022 12:08:19 +0200 (CEST)
X-Barracuda-Envelope-From: alain.berset@conf.ch
X-Barracuda-Effective-Source-IP: 00-00-000-000.dclient.hispeed.ch[00.00.000.000]
X-Barracuda-Apparent-Source-IP: 00.00.000.000
X-ASG-Quarantine-RBL: skw7uek3kd32efi5y4p4ef3mey.zen.dq.spamhaus.net
Received: from [127.0.1.1] (unknown [172.18.0.1])
	by mail.example.com (Postfix) with ESMTPA id 9AF94161F34
	for <alexandra.cerottini@heig-vd.ch>; Sun,  3 Apr 2022 06:08:18 -0400 (EDT)
Content-Type: multipart/mixed;
	boundary="===============7266181206034559405=="
MIME-Version: 1.0
From: =?utf-8?b?QWxhaW4gQmVyc2V0?= <alain.berset@conf.ch>
To: <alexandra.cerottini@heig-vd.ch>
X-Priority:
X-MSMail-Priority:
X-Barracuda-Connect: 00-00-000-000.dclient.hispeed.ch[00.00.000.000]
X-Barracuda-Start-Time: 1648980499
X-Barracuda-URL: https://quarantine.heig-vd.ch:443/cgi-mod/mark.cgi
Subject: =?utf-8?b?TGFibyBTRU4=?=
X-Virus-Scanned: by bsmtpd at heig-vd.ch
X-Barracuda-Scan-Msg-Size: 73
X-Barracuda-BRTS-Status: 1
X-ASG-Quarantine: RBL (skw7uek3kd32efi5y4p4ef3mey.zen.dq.spamhaus.net
	)
X-Barracuda-Envelope-From: alain.berset@conf.ch
X-Barracuda-Quarantine-Per-User: PER_USER
X-Barracuda-Spam-Score: 3.20
X-Barracuda-Spam-Status: No, SCORE=3.20 using global scores of TAG_LEVEL=1000.0 QUARANTINE_LEVEL=4.0 KILL_LEVEL=5.0 tests=BSF_SC5_MJ1963, FROM_EXCESS_BASE64, FROM_EXCESS_BASE64_2, MISSING_DATE, MISSING_MID, RDNS_DYNAMIC
X-Barracuda-Spam-Report: Code version 3.2, rules version 3.2.3.97098
	Rule breakdown below
	 pts rule name              description
	---- ---------------------- --------------------------------------------------
	0.14 MISSING_MID            Missing Message-Id: header
	0.01 FROM_EXCESS_BASE64     From: base64 encoded unnecessarily
	1.40 MISSING_DATE           Missing Date: header
	1.05 FROM_EXCESS_BASE64_2   From: base64 encoded unnecessarily
	0.10 RDNS_DYNAMIC           Delivered to trusted network by host with
	                           dynamic-looking rDNS
	0.50 BSF_SC5_MJ1963         Custom Rule MJ1963
Message-ID: <9722861e-8519-4d17-b6e8-bfb8595f1204@EIMAIL01.einet.ad.eivd.ch>
Return-Path: alain.berset@conf.ch
Date: Sun, 3 Apr 2022 13:31:03 +0200
X-MS-Exchange-Organization-Network-Message-Id: b365c026-d28c-411b-f92c-08da15656f71
X-MS-Exchange-Organization-AVStamp-Enterprise: 1.0
X-MS-Exchange-Organization-AuthSource: EIMAIL01.einet.ad.eivd.ch
X-MS-Exchange-Organization-AuthAs: Anonymous
X-MS-Exchange-Transport-EndToEndLatency: 00:00:00.1891825
X-MS-Exchange-Processed-By-BccFoldering: 15.01.2375.024
```

Il est intéressant de remarquer que le Barracuda Spam Score est à 3.20. Ce score varie de 0 (pas du spam) à 10 ou plus (spam). Il a pourtant été filtré. 

Nous voyons aussi les causes de ce score de 3.20 (manque de l'header, encodage en base 64, manque de la date,  dynamic-looking rDNS et la règle MJ1963 de Barracuda).

Le Return-Path correspond bien à l'adresse mail du champ From.

Nous pouvons voir le domaine (mail.example.com) ainsi que l'adresse IP.

Les champs Reply-to, X-Distribution, X-Mailer et Bcc n'existent pas.



Courrier électronique que j'ai envoyé avec SET sur une adresse mail privée (avec modifications des adresses IP) qui a été considéré comme SPAM:

```
Delivered-To: petitchat13@gmail.com
Received: by 2002:a05:6900:3344:0:0:0:0 with SMTP id y4csp95454yaa;
        Sun, 3 Apr 2022 03:11:09 -0700 (PDT)
X-Google-Smtp-Source: ABdhPJwq4c6B+5kpuCEZWDKbDEnCVAOG4dAMnMKwnYZAGLG5092IrM3JOVnRhSSIo2Eg2hHdXFt1
X-Received: by 2002:a05:600c:4e11:b0:38c:bd19:e72c with SMTP id b17-20020a05600c4e1100b0038cbd19e72cmr15421513wmq.174.1648980669844;
        Sun, 03 Apr 2022 03:11:09 -0700 (PDT)
ARC-Seal: i=1; a=rsa-sha256; t=1648980669; cv=none;
        d=google.com; s=arc-20160816;
        b=f9ZVfzpm5Db1V00QBEAOPo5p+lJuuSwuXYveM5vb2nPJJ4jE2T8JAExf1Q7+39vwWX
         AoFGQGQqg8S7A9HJwa6ygCNaClwQXQmY71ugMPjQEPx3fUXdwm9yFku8BJ/TrMoPRK/d
         pLH3YSvE4qHoMuPLUik1fqN1CscMUNqmHTHk1TXQqbLbP4VJI9LlysuNjmYu2bIOoT7U
         88QLgq3iG1OJOV0M1UMDAiyoU3xk9F7j+pSBqLkRHKP5nI+F12XBeO0eRmE6IKBIOyIc
         Q7y9ppYd5hBxRy7mZcncoNjLJN5Ha7HCEnDnDLGC+vZIUSvTwuAq0ZO8umnJoFOO6oRN
         YQIA==
ARC-Message-Signature: i=1; a=rsa-sha256; c=relaxed/relaxed; d=google.com; s=arc-20160816;
        h=subject:to:from:mime-version:message-id:date;
        bh=ZRCcAmgpajh3gR6+h181NoX+gw5R9LQK6KRTnLm/HtY=;
        b=LO/x05so2/NVwbI7Mt1DpaWBrB/4p/o7OZNh1vtZpoYyTCHg/wAOFszAQ8Pej9SVq0
         zrdlD2s6ZcDsRfHiSzMvbdspj5Tispz3CL3miALwfro7OI6/lkVmaP6w6+lYsqYiQycm
         E8y9PD4mBVKz9Fx4oCKzBuVLAv7RJeL8PdlSrO7SOXYSR821UpoiZLEo7wUJO7gCZF6p
         Ob2VBusnV01pe/oCDuhNTl46DxMoV5zQeSfWB2pkNNbgXVC/Wq1KGpY7Fx+DFUMPU0r/
         BvG1zM1MgZYqemFcTc7Ga6mbY7Xwdb6yfDbOUB9mHZA5RNESI8emojr/sqaiwwRrAVro
         1ezg==
ARC-Authentication-Results: i=1; mx.google.com;
       spf=neutral (google.com: 84.73.233.150 is neither permitted nor denied by best guess record for domain of alain.berset@conf.ch) smtp.mailfrom=alain.berset@conf.ch
Return-Path: <alain.berset@conf.ch>
Received: from mail.example.com (84-73-233-150.dclient.hispeed.ch. [84.73.233.150])
        by mx.google.com with ESMTP id r12-20020a5d52cc000000b002041b7de588si5837922wrv.770.2022.04.03.03.11.09
        for <petitchat13@gmail.com>;
        Sun, 03 Apr 2022 03:11:09 -0700 (PDT)
Received-SPF: neutral (google.com: 84.73.233.150 is neither permitted nor denied by best guess record for domain of alain.berset@conf.ch) client-ip=84.73.233.150;
Authentication-Results: mx.google.com;
       spf=neutral (google.com: 84.73.233.150 is neither permitted nor denied by best guess record for domain of alain.berset@conf.ch) smtp.mailfrom=alain.berset@conf.ch
Date: Sun, 03 Apr 2022 03:11:09 -0700 (PDT)
Message-ID: <624972bd.1c69fb81.f88a9.a113SMTPIN_ADDED_MISSING@mx.google.com>
Received: from [127.0.1.1] (unknown [172.18.0.1]) by mail.example.com (Postfix) with ESMTPA id 86A27161F34 for <petitchat13@gmail.com>; Sun,
  3 Apr 2022 06:11:08 -0400 (EDT)
Content-Type: multipart/mixed; boundary="===============1026937488467720192=="
MIME-Version: 1.0
From: Alain Berset <alain.berset@conf.ch>
To: petitchat13@gmail.com
X-Priority: 
X-MSMail-Priority: 
Subject: Labo SEN - gmail

--===============1026937488467720192==
MIME-Version: 1.0
Content-Type: text/plain; charset="utf-8"
Content-Transfer-Encoding: base64

UGV0aXQgdGVzdCBzdXIgZ21haWwK
--===============1026937488467720192==--
```

Nous pouvons remarquer que dans `Authentication-Results` le spf est **neutral**.

Le Return-Path correspond bien à l'adresse mail du champ From.

Nous pouvons voir le domaine (mail.example.com) ainsi que l'adresse IP.

Les champs Reply-to, X-Distribution, X-Mailer et Bcc n'existent pas.



Courrier électronique non-spam reçu sur une adresse mail privée (avec modifications des adresses IP):

```
Delivered-To: petitchat13@gmail.com
Received: by 2002:a05:6900:2e:0:0:0:0 with SMTP id f14csp749397yap;
        Tue, 29 Mar 2022 08:49:34 -0700 (PDT)
X-Google-Smtp-Source: ABdhPJxw1r3t3Sc3qycoKbyzPsJsRtVsd3CK1UNEqGzx5KrWlJk1OxL0MUKENhGd3VPlg0iOfWJb
X-Received: by 2002:ac8:5b50:0:b0:2eb:8756:d7c1 with SMTP id n16-20020ac85b50000000b002eb8756d7c1mr6176966qtw.378.1648568974018;
        Tue, 29 Mar 2022 08:49:34 -0700 (PDT)
ARC-Seal: i=1; a=rsa-sha256; t=1648568974; cv=none;
        d=google.com; s=arc-20160816;
        b=Tm7M1D8wlRGg1zLjnm2YZlTaDcPld6LjuXr6tfFTGcaJzK3GnXX/hlaSP02rMd33GJ
         Gev2mUy+00Ua/iW2V7TaqkbKC0P8GDIexrBRbZw6nHTHFBWdiQOnDoBMm5ag3Hbl0Kvn
         XsYezmQeKzQsJmN9GsaGpf6T8pTco+bHFtCh0atpegnkfAWemBL34BU8XnEi4PoqG9/3
         P4GKD7dIxGgba9xsDU/S1+vPmtXNr9+ortTAo6k36cavGTswIETIFPouGaWXSCqUjZdc
         CM6fCFVyQ/+U+hBihIGdyo78V+fwhaLtBepJ8xvsXtT6xRlOQTcZOhyS1mJ03VIMVeUW
         Gy+A==
ARC-Message-Signature: i=1; a=rsa-sha256; c=relaxed/relaxed; d=google.com; s=arc-20160816;
        h=feedback-id:message-id:list-id:mime-version:list-help:date:subject
         :to:from:dkim-signature;
        bh=YuxNDzOygDQLvmLjRcD5/dSFqtDd5ukbz/Q7RxLUIj0=;
        b=hADs+azb3rqDVxMPaaPaRIxSYw5+zzey+N8P95iL9paHP9D6w0E43chKCBOIS2RH7H
         sp5b+Fw9mCHePSbp7YE+bPAg6zA0WyK/mONobJ4jKjyVdxBBPPfVUMv/x3mTb8rPJh25
         lVAjv4bX1ULEtDy4EJ374BGDsjSEBscK1jigkKwKd/N4SnQk5TPZanFN+84DaH10K9IT
         iM/sDqjAqcMr15S25+Y5jTH2ATXmQvmCMqENaIkwheywqvTAbhqtcTx6JaFGtQrva8a3
         hmn5AfZCKDDeQrf0JoGsjzMosxQq3Qj5tl4y/L/4iNKrrVPqm6492kKOCZGMIziUGJpI
         CUuQ==
ARC-Authentication-Results: i=1; mx.google.com;
       dkim=pass header.i=@mail.nintendo-europe.com header.s=200608 header.b=jQ6J6emz;
       spf=pass (google.com: domain of bounce-672_html-263022483-1934766-7230703-6560@bounce.mail.nintendo-europe.com designates 13.111.14.195 as permitted sender) smtp.mailfrom=bounce-672_HTML-263022483-1934766-7230703-6560@bounce.mail.nintendo-europe.com
Return-Path: <bounce-672_HTML-263022483-1934766-7230703-6560@bounce.mail.nintendo-europe.com>
Received: from mta29.ccg.nintendo.net (mta29.ccg.nintendo.net. [13.111.14.195])
        by mx.google.com with ESMTPS id e29-20020a05620a015d00b0067e4be23a33si8790063qkn.632.2022.03.29.08.49.33
        for <petitchat13@gmail.com>
        (version=TLS1_2 cipher=ECDHE-ECDSA-AES128-GCM-SHA256 bits=128/128);
        Tue, 29 Mar 2022 08:49:34 -0700 (PDT)
Received-SPF: pass (google.com: domain of bounce-672_html-263022483-1934766-7230703-6560@bounce.mail.nintendo-europe.com designates 13.111.14.195 as permitted sender) client-ip=13.111.14.195;
Authentication-Results: mx.google.com;
       dkim=pass header.i=@mail.nintendo-europe.com header.s=200608 header.b=jQ6J6emz;
       spf=pass (google.com: domain of bounce-672_html-263022483-1934766-7230703-6560@bounce.mail.nintendo-europe.com designates 13.111.14.195 as permitted sender) smtp.mailfrom=bounce-672_HTML-263022483-1934766-7230703-6560@bounce.mail.nintendo-europe.com
DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed; s=200608; d=mail.nintendo-europe.com; h=From:To:Subject:Date:List-Help:MIME-Version:List-ID:X-CSA-Complaints: Message-ID:Content-Type; i=nintendo@mail.nintendo-europe.com; bh=YuxNDzOygDQLvmLjRcD5/dSFqtDd5ukbz/Q7RxLUIj0=; b=jQ6J6emzlLnEX+Mfh9y9/Dwoz5QdArPkm9hvTzPWXsJo6sgJOYRcsak3LTL/lbMQXXtTFfzUdY9R
   xvCQk3cmqOcXGWQZQNsHm/yUEqdsWeYutz5h+GWiQq6bke8Hi7sNa2nlpqo8SrQBaB+xQ9RvLaeY
   xj5YSj6Q+nbmO7i5jA4=
Received: by mta29.ccg.nintendo.net id h8cl8q2fmd40 for <petitchat13@gmail.com>; Tue, 29 Mar 2022 15:49:31 +0000 (envelope-from <bounce-672_HTML-263022483-1934766-7230703-6560@bounce.mail.nintendo-europe.com>)
From: Nintendo <nintendo@mail.nintendo-europe.com>
To: <petitchat13@gmail.com>
Subject: Informations au sujet des changements liés à l'entrepôt dans Animal Crossing: Pocket Camp
Date: Tue, 29 Mar 2022 09:49:31 -0600
List-Help: <https://click.ccg.nintendo.com/subscription_center.aspx?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJtaWQiOiI3MjMwNzAzIiwicyI6IjI2MzAyMjQ4MyIsImxpZCI6IjY3MiIsImoiOiIxOTM0NzY2IiwiamIiOiI2NTYwIiwiZCI6IjcwMTgwIn0.YOKTMB9Z1CwTK5xlnP_ZJoCgasjmaQig4ZL5S41gbS0>
x-CSA-Compliance-Source: SFMC
MIME-Version: 1.0
List-ID: <7207004.xt.local>
X-CSA-Complaints: csa-complaints@eco.de
X-SFMC-Stack: 7
x-job: 7230703_1934766
Message-ID: <cb166d8f-f646-4ebe-b654-039adb4001b0@atl1s07mta2959.xt.local>
Feedback-ID: 7230703:1934766:13.111.14.195:sfmktgcld
Content-Type: multipart/alternative; boundary="8WqCrxziorSi=_?:"

--8WqCrxziorSi=_?:
Content-Type: text/plain; charset="utf-8"
Content-Transfer-Encoding: 8bit


***********************************************************************
Tous les joueurs vont bientôt pouvoir utiliser l'entrepôt gratuitement !
***********************************************************************
Fan de Nintendo, bonjour !

Votre messagerie électronique ne supporte malheureusement 
pas les e-mails au format HTML. Il est possible que la prise en 
charge d'e-mails au format HTML soit désactivée dans les
paramètres de sécurité de votre messagerie.

Pour visionner le contenu de cette newsletter Nintendo, cliquez 
sur le lien ci-dessous :

https://view.ccg.nintendo.com/?qs=b3db05e17b235493ee593a75b7e50dd8887ac882ff43301a24091a0ffc216694f407820e0a7ee6e23faefd58e67bd65fed7aa3b57fa6951c03dc46edad2800b1ca49c96fef93ccdd9bbea4978d1cfab9 


***********************************************************************
© 2022 Nintendo Co., Ltd.| Nintendo 3DS, Wii U et Nintendo Switch sont des marques déposées de Nintendo.
***********************************************************************

{...}
```

Nous pouvons remarquer que dans `Authentication-Results` le spf est **pass**. Le domaine est donc considéré comme un domaine permis.

Le Return-Path ne correspond pas à l'adresse mail du champ From. Ceci est sûrement dû au fait que le mail est une Newsletter.

Nous pouvons voir le domaine (mta29.ccg.nintendo.net) ainsi que l'adresse IP.

Les champs Reply-to, X-Distribution, X-Mailer et Bcc n'existent pas.

---
#### Partagez avec nous vos conclusions.

```
Conclusions :
Il est intéressant de créer son propre serveur mail. Nous voyons à quel point il peut être facile de se faire passer pour quelqu'un d'autre. Malheureusement, il faut tout de même faire attention à ce que le mail ne se retrouve pas dans les spams. 
L'outil SET est très pratique et il propose énormément d'options. On voit à quel point il est simple de copier une page web et de récupérer des identifiants par exemple.
```
---

## Echeance

Le 14 avril 2022 à 10h25
