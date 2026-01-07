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
 - Il vous faut une PAC compatible avec le BSB LAN. Voici ici la liste des appareils compatibles : 

## Les étapes que j'ai suivies de mon côté
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

### Configuration de Home Assistant

J'ai suivi cet excellent Github : https://github.com/ryann72/Home-assistant-tutoriel/blob/main/BSB-LAN/tutoriel%20BSB-LAN.md

Là, par contre, j'ai davantage de choses à vous raconter. Pour la configuration de Home Assistant, j'ai procédé à plusieurs étapes :
 1. J'ai installé le module **File Editor** (Dans **Paramètres > Modules Complémentaires**) ainsi que **Mosquitto Broker**
 2. Une fois File Editor installé, il faut le démarrer et j'ai coché la case **Ajouter à la barre latérale** pour y accéder plus rapidement
 3. A l'aide de File Editor, j'ai créé plusieurs fichiers :
     - **configuration.yaml** : permet de configurer Home Assistant
     - **mqtt.yaml** : permet de gérer la communication vers le BSB
     - **button.yaml** : permet de créer des boutons dans l'interface Home Assistant
     - **climate.yaml** : permet de gérer les thermostats
     - **select.yaml** : permet de créer des listes déroulantes
     - **switch.yaml** : permet de créer des boutons également qui me seront utiles ensuite avec Google Home
     - **service_account.json** : contient les informations pour communiquer avec Google Cloud

Je vous mets tous mes fichiers ici : https://github.com/vincent2mots/domotique/tree/main/home_assistant

**Attention au fichier mqtt.yaml** : il contient les informations de communication vers ma PAC mais si votre PAC est différente, il vous faudra adapter ce fichier (et les autres aussi sans doute!)