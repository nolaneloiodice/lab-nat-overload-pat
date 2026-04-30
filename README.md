# Lab réseau - NAT Overload / PAT

## Objectif

Mettre en place le NAT Overload, aussi appelé PAT, afin de permettre à un réseau LAN privé d’accéder à un réseau externe simulant Internet.

Ce lab permet de comprendre comment une machine avec une adresse IP privée peut communiquer avec un réseau externe en utilisant l’adresse IP publique du routeur NAT.

---

## Topologie réseau

- 1 PC LAN
- 1 switch
- 1 routeur NAT : R1
- 1 routeur Internet simulé : R2
- 1 PC Internet configuré en 8.8.8.8

Schéma du réseau :

![Topologie réseau](topologie-nat.png)

---

## Plan d’adressage

### Réseau LAN

- PC LAN : 192.168.1.10 /24
- Passerelle : 192.168.1.1
- Routeur R1 côté LAN : 192.168.1.1

### Réseau entre R1 et R2

- R1 côté WAN : 200.0.0.1
- R2 côté R1 : 200.0.0.2

### Réseau Internet simulé

- R2 côté Internet : 8.8.8.1
- PC Internet : 8.8.8.8
- Passerelle du PC Internet : 8.8.8.1

---

## Contexte et fonctionnement

Dans ce lab, le routeur R1 joue le rôle de routeur NAT.

Le PC LAN possède une adresse IP privée en 192.168.1.0/24. Cette adresse ne doit pas être utilisée directement pour communiquer avec le réseau externe.

Le NAT permet donc de traduire l’adresse IP privée du PC LAN en utilisant l’adresse IP de l’interface WAN du routeur R1.

Dans ce lab, le NAT utilisé est du NAT Overload / PAT.  
Cela signifie que plusieurs machines internes pourraient partager la même adresse IP externe grâce à l’utilisation des ports.

---

## Configuration de R1 - Routeur NAT

### 1. Configuration des interfaces

```bash
enable
configure terminal

interface gigabitethernet 0/0/0
ip address 192.168.1.1 255.255.255.0
ip nat inside
no shutdown
exit

interface gigabitethernet 0/0/1
ip address 200.0.0.1 255.255.255.0
ip nat outside
no shutdown
exit
L’interface côté LAN est déclarée en ip nat inside.

L’interface côté réseau externe est déclarée en ip nat outside.

2. Création de l’ACL NAT
access-list 1 permit 192.168.1.0 0.0.0.255

Cette ACL indique au routeur quel réseau interne a le droit d’être traduit par le NAT.

Ici, seul le réseau 192.168.1.0/24 est concerné.

3. Activation du NAT Overload
ip nat inside source list 1 interface gigabitethernet 0/0/1 overload

Cette commande indique que les adresses autorisées par l’ACL 1 seront traduites en utilisant l’adresse IP de l’interface WAN de R1.

Le mot-clé overload permet à plusieurs machines internes de partager la même adresse IP externe.

4. Route par défaut vers Internet simulé
ip route 0.0.0.0 0.0.0.0 200.0.0.2

Cette route permet à R1 d’envoyer vers R2 tout le trafic destiné à un réseau qu’il ne connaît pas.

Configuration de R2 - Routeur Internet simulé
enable
configure terminal

interface gigabitethernet 0/0/0
ip address 200.0.0.2 255.255.255.0
no shutdown
exit

interface gigabitethernet 0/0/1
ip address 8.8.8.1 255.255.255.0
no shutdown
exit

R2 simule un routeur situé sur Internet.

Il permet de relier le réseau WAN entre R1 et R2 au réseau du PC Internet en 8.8.8.0/24.

Tests de connectivité
Ping depuis le PC LAN vers 8.8.8.8

Le PC LAN arrive à joindre le PC Internet configuré en 8.8.8.8.

Ce test valide que :

le PC LAN utilise bien sa passerelle
R1 route correctement le trafic
le NAT est actif
R2 permet d’atteindre le réseau Internet simulé
Vérification du NAT

Commande utilisée sur R1 :

show ip nat translations

Résultat :

La table NAT montre que l’adresse privée du PC LAN est traduite par le routeur R1.

Cela confirme que le NAT Overload fonctionne correctement.

Méthode de diagnostic utilisée

Pour vérifier le fonctionnement du lab, j’ai utilisé la méthode suivante :

observation → tests → résultats → hypothèses → correction → validation

Exemples de raisonnement :

Si le PC LAN ne ping pas sa passerelle → problème local
Si le PC LAN ping R1 mais pas R2 → problème de routage ou d’interface WAN
Si le PC LAN ping R2 mais pas 8.8.8.8 → problème de routage côté Internet simulé
Si le ping vers 8.8.8.8 fonctionne mais que show ip nat translations reste vide → problème de configuration NAT
Résultat

Le PC LAN peut communiquer avec le PC Internet en 8.8.8.8 grâce au NAT Overload configuré sur R1.

Le lab est fonctionnel et permet de comprendre le rôle du NAT dans l’accès d’un réseau privé à un réseau externe.

Compétences développées
Configuration du NAT Overload / PAT
Différence entre réseau interne et réseau externe
Utilisation de ip nat inside et ip nat outside
Création d’une ACL pour identifier le trafic à traduire
Configuration d’une route par défaut
Vérification avec show ip nat translations
Diagnostic réseau avec ping et commandes Cisco IOS
Fichiers
lab-nat-overload-pat.pkt
topologie-nat.png
ping-lan-vers-8.8.8.8.png
show-ip-nat-translations.png