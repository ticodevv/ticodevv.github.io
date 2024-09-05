---
# type: docs 
title: Probe Attack
date: 2024-08-20 00:00:00+0000  
featured: true
draft: false
comment: true
toc: true
reward: true
pinned: false
series:
  - News
categories:
  - Hardware
tags: 
  - WiFi
  - ESP32
  - Probe Attack
  - Probe Sniffing
authors:
  - Lukas
images: []
---

![M5stack Cardputer](image-1.png)

Je fais ce post suite à la découverte d'un projet qui m'a beaucoup plus : l'EVIL-M5Core2. Dans ce projet, il y a une attaque que je ne connaissais pas et qui m'a beaucoup intéressé : la Probe Attack. Comme je ne connaissais pas cette attaque, je me suis dit que je devais la partager avec vous.

<!--more-->


`Exemple` :

Lorsque vous allumez le WiFi sur votre smartphone, il envoie des Probe Requests pour rechercher les réseaux disponibles autour de vous. Ces requêtes contiennent souvent le nom (SSID) des réseaux auxquels l'appareil s'est déjà connecté dans le passé, afin de voir si l'un de ces réseaux est disponible.

`Rôle des Probe Requests` :

- **Découverte des réseaux** : Les Probe Requests permettent à un appareil de découvrir quels réseaux WiFi sont disponibles dans son environnement immédiat.
- **Réponse des bornes** : Les points d'accès WiFi (comme votre routeur à la maison) qui reçoivent ces requêtes peuvent répondre avec des Probe Responses, indiquant ainsi à l'appareil qu'un réseau est disponible.

## Qu'est-ce que la Probe Sniffing ?

Le Probe Sniffing est une technique utilisée pour intercepter les Probe Requests envoyées par les appareils WiFi. En capturant ces requêtes, un attaquant peut obtenir des informations sur les réseaux auxquels un appareil s'est déjà connecté, y compris les SSID (noms des réseaux) et les adresses MAC des points d'accès.

### Comment fonctionne la Probe Sniffing ?

`Exemple` :
- Un appareil qui a le WiFi activé et qui n'est pas connecté à un réseau va émettre des Probe Requests pour rechercher les réseaux disponibles autour de lui.
- Quand on rentre chez soi et que l'on arrive dans le salon, notre smartphone va émettre des Probe Requests pour chercher notre réseau WiFi et se reconnecter automatiquement.
- C'est parce que le téléphone ou l'appareil a probablement crié le nom de notre réseau WiFi dans les Probe Requests toute la journée.

On peut récupérer ces Probe Requests pour les analyser ou réaliser une attaque de type Karma.

### Schéma illustratif

```text
[Appareil émet une Probe Request]  -->  [Sniffer capture la Probe Request]  -->  [Extraction des données]  -->  [Analyse]  -->  [Exploitation des données]
      (Adresse MAC, SSID, etc.)         (Adresse MAC, SSID, etc.)            (Localisation, Habitudes)       (Surveillance, Marketing, Attaques)
```

## La Probe Attack

On vient de voir que les appareils envoient des Probe Requests pour rechercher les réseaux WiFi autour d'eux. Mais que peut-on faire de ces Probe Requests ?

1. **Collecte d'informations sensibles**

   Les Probe Requests contiennent des informations comme les noms (SSID) des réseaux WiFi auxquels un appareil s'est déjà connecté. Ces informations peuvent être utilisées pour :

   - **Suivi des utilisateurs** : En capturant les Probe Requests, un attaquant peut suivre les déplacements d'une personne en analysant les réseaux WiFi auxquels elle tente de se connecter dans différents lieux.
   - **Ciblage personnalisé** : Connaître les SSID des réseaux WiFi précédemment utilisés par une victime peut permettre à un attaquant de mieux comprendre les habitudes de la personne, comme les lieux qu'elle fréquente.

### Schéma illustratif
  
  ```text
[Appareil émet une Probe Request]  -->  [Attaquant capture la Probe Request]  -->  [Attaquant exploite les données]  -->  [Émission d'une fausse réponse]  -->  [Appareil se connecte à un faux réseau]
      (Adresse MAC, SSID, etc.)            (Adresse MAC, SSID, etc.)              (Création d'un faux réseau Wi-Fi)      (Attaque de type Man-in-the-Middle)
  ```
  
2. **Attaque Evil Twin**

   Une attaque de type Evil Twin consiste à créer un faux point d'accès WiFi avec le même nom qu'un réseau auquel la victime a déjà tenté de se connecter (SSID capturé dans les Probe Requests). L'appareil de la victime pourrait automatiquement se connecter à ce faux réseau, croyant qu'il s'agit du réseau légitime. Une fois connecté :

   - **Interception du trafic** : L'attaquant peut surveiller et intercepter tout le trafic Internet de la victime, y compris les informations sensibles comme les identifiants et mots de passe.
   - **Attaque de l'homme du milieu (MitM)** : En se plaçant entre l'appareil de la victime et Internet, l'attaquant peut modifier ou injecter des données dans les communications de la victime.

3. **Attaque de type Karma**

   Dans une attaque de type Karma, l'attaquant configure un point d'accès WiFi pour répondre automatiquement à toute Probe Request qu'il reçoit, prétendant être n'importe quel réseau recherché par les appareils. Voici comment cela fonctionne :

   - **Réponse aux Probe Requests** : Lorsque les appareils envoient des Probe Requests, le point d'accès de l'attaquant répond en se faisant passer pour le réseau recherché.
   - **Connexion automatique des appareils** : Si un appareil reconnaît le SSID annoncé par l'attaquant, il peut se connecter automatiquement au réseau de l'attaquant, croyant qu'il s'agit du réseau légitime.

   Une fois connecté, l'attaquant peut alors réaliser des actions similaires à celles de l'attaque Evil Twin, telles que l'interception du trafic ou l'accès à des données sensibles.

4. **Attaque par usurpation d'identité**

   Les adresses MAC (Media Access Control) des appareils peuvent également être interceptées via les Probe Requests. Avec cette information, un attaquant peut :

   - **Usurper l'identité d'un appareil** : En utilisant l'adresse MAC d'un autre appareil, l'attaquant peut se faire passer pour cet appareil sur le réseau WiFi, potentiellement pour accéder à des ressources réseau réservées.