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