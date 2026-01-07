# Tuto de domotisation d'une PAC Atlantic Alfea non IO et intégration dans Google Home

## Préambule
### L'origine de ce projet
Ma pompe à chaleur (PAC) est un modèle de 2012 non IO, c'est à dire non domotisable nativement. Il s'agit d'une PAC Atlantic Alfea Extensa Duo Interver. Pendant des années, je me suis laissé convaincre qu'il me serait impossible de la domotiser afin de pouvoir notamment la contrôler à distance, augmenter / baisser les températures, mettre en place des automatisations intelligentes, etc.

Puis je suis tombé sur un blog incroyable (https://www.sheevaboite.fr/articles/domotiser-pompe-chaleur-atlantic-alfea-bsb-lan/) qui m'a fait changer d'avis.

A priori, c'était maintenant possible! Je me suis donc lancé dans ce projet avec un objectif simple : **je souhaitais pouvoir intégrer dans mon Google Home des éléments de contrôle de ma PAC, comme le mode de chauffe ou la consigne de température**.

Et et cerise sur le gâteau : **je souhaitais une solution libre et gratuite (donc sans utiliser Nabu Casa)**

### Disclaimer
Avant toute chose, je raconte ici les différentes étapes pour parvenir à cette domotisation mais je ne suis évidemment pas responsable si, en tentant chez vous, cela ne fonctionnait pas.

### Schéma final de communication

Voici le schéma final de mon installation : 

![image](https://raw.githubusercontent.com/vincent2mots/domotique/refs/heads/main/images/schema.png)

**Ca peut faire peur mais si on résume :**
 1. La PAC dans laquelle j'ai installé le BSB LAN branché sur le microcontrôleur OLIMEX ESP32 POE ISO
 2. IL y a mon NAS qui me permet deux fonctionnalités importantes ici :
     1. *Il me permet de virtualiser une machine contenant Home Assistant et le broker Mosquitto qui va communiquer en lecture / écriture avec le combo Olimex / BSB*
     2. *Il dispose d'un reverse proxy qui va me permettre d'exposer Home Assistant sur Internet, pour pouvoir ensuite le lier avec Google Cloud*
 3. Il y a ma Freebox Ultra qui me permet surtout de faire de la redirection de ports du domaine Internet vers mon réseau local (pour exposer correctement Home Assistant sur le web)
 4. Enfin il y a la partie Google Cloud et Google Home qui me permet de piloter ma PAC (et d'autres appareils) depuis mon smartphone, à l'aide de l'application Google Home


### Liens utiles
 - Le blog qui m'a permi de commencer la domotisation : https://www.sheevaboite.fr/articles/domotiser-pompe-chaleur-atlantic-alfea-bsb-lan/
  - La super documentation de l'installation du BSB-LAN : https://docs.bsb-lan.de/fr/index.html
  - Ce github très utile également sur la configuration côté Home Assistant surtout : https://github.com/ryann72/Home-assistant-tutoriel/blob/main/BSB-LAN/tutoriel%20BSB-LAN.md
   - Enfin, cette documentation très bonne aussi concernant l'intégration de Home Assistant dans Google Home : https://www.home-assistant.io/integrations/google_assistant/

### Prérequis :
Si on regarde le schéma précédent, voici les prérequis de mon côté :
 - Quelques compétences en informatique / réseau / virtualisation / etc.
 - L'acquisition du BSB LAN et du microcontrôleur
 - Un NAS ou n'importe quel système capable de contenir une VM Home Assistant (ou un boitier de chez Nabu Casa si ma configuration vous effraie)
 - Un reverse proxy (Mon NAS en proposait un mais ça dépend des NAS...)
 - Un nom de domaine (j'ai utilisé celui proposé par mon NAS Synology)
 - Il vous faut une PAC compatible avec le BSB LAN. Voici ici la liste des appareils compatibles : https://docs.bsb-lan.de/fr/supported_heating_systems.html

## Lot 1 : piloter ma PAC depuis mon réseau local avec Home Assistant
### Achat du BSB LAN et du micro contrôleur
Le BSB LAN ne s'achète pas dans le commerce. Si vous en voulez un, il faut envoyer un mail à Frederik Holst (bsb@code-it.de) en anglais ou en allemand. L'appareil devrait vous coûter 50 €.

Ensuite, il vous faut un microcontôleur. Plusieurs options sont possibles (ESP32 ou Arduino). De mon côté, j'ai opté pour celui-ci : https://www.olimex.com/Products/IoT/ESP32/ESP32-POE-ISO/open-source-hardware
Avec l'envoi, le coût passe à 46 € environ.

**On est donc sur un coût inférieur à 100 € tout compris**

### Installation du BSB LAN et du micro contrôleur
Ma PAC dispose d'une carte Siemens AVS55. J'ai donc branché le BSB LAN à l'emplacement indiqué : 
![image](https://raw.githubusercontent.com/vincent2mots/domotique/refs/heads/main/images/carte_siemens_AvS55.jpg)

Voici l'image complète de l'installation : 
![image](https://raw.githubusercontent.com/vincent2mots/domotique/refs/heads/main/images/installation_bsb.jpg)

le micro contrôleur est alimenté par du micro USB avec un chargeur de téléphone.

### Configuration du micro contrôleur
Pour cette partie, je ne vais pas refaire la documentation officielle du BSB LAN qui est parfaite et très claire : https://docs.bsb-lan.de/fr/install.html#assemblage-de-ladaptateur-bsb-lan

Mais je résume les étapes :
 1. Il va falloir installer l'IDE Arduino sur votre poste et cloner le GIT de BSB LAN pour récupérer toutes les infos
 2. Il va falloir connecter le micro contrôleur à votre PC pour y charger le programme (on parle ici de flasher la carte). Attention, cela ne fonctionnait pas chez moi dans un premier temps car le câble que j'avais utilisé ne transmettait pas de data! Attentio à ce point
 3. Une fois le programme chargé, on installe le tout dans la PAC
 4. On peut alors visiter le site local : http://bsb-lan.local
 5. Ici, il faut alors télécharger la liste des paramètres spécifiques à l'appareil et envoyer le fichier à Frederik à nouveau. Il vous renverra un fichier et il faudra à nouveau flasher la carte avec l'IDE Arduino
 6. C'est fini!

### Premiers tests avec l'URL bsb-lan.local
Une fois le tout installé et configuré, il devient possible de piloter la PAC depuis le site local : http://bsb-lan.local

Attention, au début, je ne parvenais pas à modifier les valeurs mais seulement à les afficher. Pour pouvoir les modifier, il faut se rendre dans **Paramètres** et modifier la valeur **Général > Accès en écriture (niveau)** à **Activé (complet)**. Sinon vous serez uniquement en lecture seule.

### Mise en place de Home Assistant
A nouveau, je ne vais rien inventer de plus que ce blog qui résume très bien comment installer une VM Home Assistant sur un NAS Synology (mon cas) : https://www.cachem.fr/synology-home-assistant-vm/

Si cela vous effraie, sachez qu'il est possible d'acheter une BOX domotique toute prête dans laquelle Home Assistant est déjà installé : https://www.domadoo.fr/fr/produits-compatibles-home-assistant/7046-nabu-casa-box-domotique-home-assistant-green-0860011789703.html?srsltid=AfmBOoqawa1pB0I6udh0El10Sf2ktNsAlTy-ikXN4vOSJvcocsIjfeRV

### Communication entre BSB et Home Assistant
J'ai suivi cet excellent Github : https://github.com/ryann72/Home-assistant-tutoriel/blob/main/BSB-LAN/tutoriel%20BSB-LAN.md

#### Configuration Home Assistant

Là, par contre, j'ai davantage de choses à vous raconter. Pour la configuration de Home Assistant, j'ai procédé à plusieurs étapes :
 1. J'ai installé le module **File Editor** (Dans **Paramètres > Modules Complémentaires**) ainsi que **Mosquitto Broker**
 2. Une fois File Editor installé, il faut le démarrer et j'ai coché la case **Ajouter à la barre latérale** pour y accéder plus rapidement
 3. A l'aide de File Editor, j'ai créé plusieurs fichiers :
     - **configuration.yaml** : permet de configurer Home Assistant
     - **mqtt.yaml** : permet de gérer la communication vers le BSB
     
Je vous mets tous mes fichiers ici : https://github.com/vincent2mots/domotique/tree/main/home_assistant

**Attention au fichier mqtt.yaml** : il contient les informations de communication vers ma PAC mais si votre PAC est différente, il vous faudra adapter ce fichier (et les autres aussi sans doute!)

### Configuration de MQTT dans BSB
Pour pouvoir lire / écrire avec Home Assistant, il faut configurer MQTT côté BSB LAN pour pouvoir communiquer. Cela se fait dans **Paramètres**

Il faut d'abord activer le paramètre général **Afficher les réglages avancés**

Ensuite, on peut configurer la partie MQTT : 
![image](https://raw.githubusercontent.com/vincent2mots/domotique/refs/heads/main/images/mqtt_bsb_lan.png)

Dans la partie **Paramètres**, j'ai mis tous les paramètres de la PAC qui m'intéressaient personnellement. Dans mon cas, c'est la liste suivante : 
```
700,710,712,8740,1000,1010,1012,1600,8700,8830,8411,8412,8410,8000,8001,8003,8730,8821,1,2,3
```

### La partie visuelle dans Home Assistant 
Toujours à l'aide de File Editor, j'ai créé les fichiers suivants :
- **button.yaml** : permet de créer des boutons dans l'interface Home Assistant
- **climate.yaml** : permet de gérer les thermostats
- **select.yaml** : permet de créer des listes déroulantes
- **switch.yaml** : permet de créer des boutons également qui me seront utiles ensuite avec Google Home
- **service_account.json** : contient les informations pour communiquer avec Google Cloud (voir lot 2)

Voici le résultat final côté Home Assistant : 
![image](https://raw.githubusercontent.com/vincent2mots/domotique/refs/heads/main/images/HA_dashboard.png)

Le bouton de synchronisation de la date / heure vient d'une problématique de mon côté : lorsque la PAC subit une coupure de courant trop longue, à son démarrage, elle perd la date du jour et je suis obligé de la resaisir manuellement. 

Le but ici était également de mettre en place côté Google Home une routine quotidienne qui viendrait remettre la bonne date du jour, pour éviter ce genre de soucis.

## Lot 2 : aller plus loin et piloter le tout depuis Google Home
### Exposition de Home Assistant sur Internet
#### Etape 1 : Configurer Home Assistant

Dans le fichier **configuration.yaml**, il faut bien s'assurer d'avoir les paramètres suivants de renseignés : 
``` yaml
homeassistant:
  external_url: "https://ha.xxxxxxxxxxxx.synology.me:60443" # Indiquer le port, si ce n'est pas du 443 (HTTPS)
  internal_url: "http://192.168.x.xxx:8123" # adresse locale de Home Assistant avec le port
  time_zone: Europe/Paris

http:
  use_x_forwarded_for: true
  trusted_proxies: # adresse(s) IP locale(s) du NAS Synology
    - 192.168.x.xxx
  ip_ban_enabled: false
  
# Ajout de la possibilité de se connecter avec un téléphone
mobile_app:
```
Ensuite, j'ai fixé dans Home Assistant l'IP de la VM, pour ne pas avoir de soucis par la suite. Dans **Paramètres > Système > Configurer les interfaces réseau > IPV4**

Puis on vérifie les YAML et on redémarre le tout dans **Outils de développement**

#### Etape 2 : Configurer le reverse proxy
Aller dans **Panneau de configuration > Portail de connexion > Avancé** pour régler le reverse proxy :
![image](https://raw.githubusercontent.com/vincent2mots/domotique/refs/heads/main/images/reverse_proxy_1.png)

Créer une règle comme ceci : 
![image](https://raw.githubusercontent.com/vincent2mots/domotique/refs/heads/main/images/reverse_proxy_2.png)
Quelques infos :
 - La partie source est ce qui sera visible depuis Internet. Mon Home Assistant sera donc accessible depuis l'URL https://ha.xxxx.synology.me:99999
 - La partie destination c'est la partie interne à mon réseau. Le nom d'hôte c'est l'IP de mon Home Assistant et le port c'est le 8123 (port par défaut côté Home Assistant, que je n'ai pas changé ici)

 L'URL source devra être reprise dans le fichier **configuration.yaml** vu ci-avant dans external_url.

 Ne pas oublier les en-têtes pour que le tout fonctionne : 
 ![image](https://raw.githubusercontent.com/vincent2mots/domotique/refs/heads/main/images/reverse_proxy_3.png)

Enfin, pour que le tout fonctionne, il faut produire un certificat qui va permettre de sécuriser vos échanges (la sécurité avant tout). Cela se fait dans **Panneau de configuration > Sécurité > Certificat**. J'ai pour ma part utilisé les certificats Let's Encrypt qui sont gratuits : 
![image](https://raw.githubusercontent.com/vincent2mots/domotique/refs/heads/main/images/certificat.png)

#### Etape 3 : Rediriger les ports
Maintenant que la partie reverse proxy est faite, il faut que votre box Internet fasse le mapping entre ce qui arrive d'Internet et votre réseau interne en faisant une redirection de port.

De mon côté, j'ai une freebox Ultra. Je me suis donc rendu sur l'URL suivante pour gérer ma box : http://mafreebox.freebox.fr/

Là, je me suis rendu dans **Paramètres de la freebox > Gestion des ports**
![image](https://raw.githubusercontent.com/vincent2mots/domotique/refs/heads/main/images/freebox_1.png)

![image](https://raw.githubusercontent.com/vincent2mots/domotique/refs/heads/main/images/freebox_2.png)

![image](https://raw.githubusercontent.com/vincent2mots/domotique/refs/heads/main/images/freebox_3.png)

### Intégration à Google Home