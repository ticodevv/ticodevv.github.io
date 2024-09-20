---
title: "Active Directory les bases"
date: 2024-09-17 00:00:00
author: "Lukas"
layout: post
permalink: /Active-Directory/
disqus_identifier: 0000-0000-0000-0000
cover: assets/uploads/2024/2024-doc-active_directory/image.png
description: ""
tags:
  - "Active Directory"
---


Bonjour à tous, dans cet article, nous allons voir les bases de l'Active Directory.
Le terme Active Directory est souvent utilisé dans le monde de l'informatique, mais qu'est-ce que c'est exactement ? Comment fonctionne-t-il ? Quels sont ses avantages et ses inconvénients ? Nous allons répondre à toutes ces questions dans cet article.

<!--more-->

## Introduction

Tout d'abord on va commencer par la base des bases, c'est quoi l'Active Directory ?

- Développé par **Microsoft**
- Système d'exploitation **Windows Server**
- Environnement **X86, x86-64 et IA-64**
- Type 	**Annuaire**
- Première apparition	**1996**
- Première utilisation	**Windows 2000 Server Édition** 
- Dernière version stable	**Windows Server 2019** (à l'heure ou j'écris cet article)
- Derrière version annoncée	**Windows Server 2025** (à l'heure ou j'écris cet article)

---
> **_Wikipedia:_** Active Directory (AD) est la mise en œuvre par Microsoft des services d'annuaire LDAP pour les systèmes d'exploitation Windows. 

---

OK cool mais c'est quoi un annuaire LDAP ?

Un annuaire LDAP c'est un service qui permet de stocker des informations sur des utilisateurs, des groupes, des ordinateurs, des imprimantes, etc. Ces informations sont organisées en arborescence et peuvent être consultées et modifiées par des applications ou des utilisateurs autorisés.

Donc en gros l'Active Directory c'est un service qui permet de stocker des informations sur les utilisateurs.

## Structure de l'Active Directory

![Active Directory](/assets/uploads/2024/2024-doc-active_directory/structur_ad.png)

**BON** C'est un beau merdier tout ça, mais en gros on a :

- **Forest** : C'est l'ensemble des domaines et des arbres qui partagent une relation de confiance.
- **Tree** : C'est un ensemble de domaines qui partagent une relation de confiance.
- **Domains** : C'est une unité d'organisation qui regroupe des objets (utilisateurs, groupes, ordinateurs, etc.) et qui partage une base de données Active Directory.
- **Organizational Unit (OU)** : C'est une unité d'organisation qui permet de regrouper des objets (utilisateurs, groupes, ordinateurs, etc.) dans un domaine.

Pour être plus clair:

- **Forest** : On peut le voir comme un continent.
- **Tree** : On peut le voir comme un pays.
- **Domains** : On peut le voir comme une ville.
- **Organizational Unit (OU)** : On peut le voir comme un quartier.

On peut aussi avoir des **Trust** entre les domaines, c'est à dire que les domaines peuvent partager des ressources entre eux. comme si on avait un pont entre deux villes. 

## Avantages de l'Active Directory

- **Centralisation des informations** : Toutes les informations sur les utilisateurs, les groupes, les ordinateurs, etc. sont stockées dans (un seul endroit) il est préférable d'avoir une redondance des données pour éviter les pertes de données en cas de panne.
- **Gestion centralisée des utilisateurs** : Les administrateurs peuvent gérer les utilisateurs, les groupes, les ordinateurs, etc. à partir d'une seule console.
- **Sécurité** : L'Active Directory permet de définir des politiques de sécurité pour les utilisateurs, les groupes, les ordinateurs, etc. Attention au misconfigurations !  

## Inconvénients de l'Active Directory

- **Complexité** : L'Active Directory est un système complexe qui nécessite une expertise pour être correctement configuré et géré.
- **Coût** : L'Active Directory est un produit payant qui nécessite l'achat de licences pour être utilisé.
- **Dépendance à Microsoft** : L'Active Directory est un produit de Microsoft qui nécessite l'utilisation de systèmes d'exploitation Windows Server.


## Conclusion

L'Active Directory est un service d'annuaire LDAP développé par Microsoft pour les systèmes d'exploitation Windows. Il permet de stocker des informations sur les utilisateurs, les groupes, les ordinateurs, etc. de manière centralisée. L'Active Directory offre de nombreux avantages, tels que la centralisation des informations, la gestion centralisée des utilisateurs et la sécurité. Cependant, il présente également des inconvénients, tels que sa complexité, son coût et sa dépendance à Microsoft. En conclusion, l'Active Directory est un outil puissant qui peut simplifier la gestion des ressources informatiques dans une entreprise, mais qui nécessite une expertise pour être correctement configuré et géré.

## sources

- [Wikipedia](https://fr.wikipedia.org/wiki/Active_Directory)

- [Microsoft](https://learn.microsoft.com/fr-fr/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview)

- [securdi](https://securdi.com/authentication/introduction-to-active-directory/)