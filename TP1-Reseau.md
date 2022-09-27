# TP1 - Mise en jambes

# I. Exploration locale en solo

## 1. Affichage d'informations sur la pile TCP/IP locale

### A - En ligne de commande

#### - Affichez les infos des cartes réseau de votre PC

```bash
# nom, adresse MAC et adresse IP de l'interface WiFi

PS C:\Users\33785> ipconfig /all

Configuration IP de Windows

[...]

Carte réseau sans fil Wi-Fi :

   Suffixe DNS propre à la connexion. . . :
   # nom de l'interface WiFi
   Description. . . . . . . . . . . . . . : Intel(R) Wi-Fi 6 AX200 160MHz
   # adresse MAC de l'interface WiFi
   Adresse physique . . . . . . . . . . . : E0-D4-E8-F2-46-9E
   DHCP activé. . . . . . . . . . . . . . : Oui
   Configuration automatique activée. . . : Oui
   Adresse IPv6 de liaison locale. . . . .: fe80::7938:13dc:dac0:406%19(préféré)
   # adresse IP de l'interface WiFi
   Adresse IPv4. . . . . . . . . . . . . .: 10.33.19.221(préféré)
   Masque de sous-réseau. . . . . . . . . : 255.255.252.0
   Bail obtenu. . . . . . . . . . . . . . : lundi 26 septembre 2022 12:01:03
   Bail expirant. . . . . . . . . . . . . : mercredi 28 septembre 2022 09:01:41
   Passerelle par défaut. . . . . . . . . : 10.33.19.254
   Serveur DHCP . . . . . . . . . . . . . : 10.33.19.254
   IAID DHCPv6 . . . . . . . . . . . : 165729512
   DUID de client DHCPv6. . . . . . . . : 00-01-00-01-27-E6-CA-8A-00-6F-00-00-0A-DF
   Serveurs DNS. . .  . . . . . . . . . . : 8.8.8.8
                                       8.8.4.4
                                       1.1.1.1
   NetBIOS sur Tcpip. . . . . . . . . . . : Activé
```

```bash
# nom, adresse MAC et adresse IP de l'interface Ethernet

Pas d'interface Ethernet
```

#### - Affichez votre gateway

```bash
PS C:\Users\33785> ipconfig

[...]

Carte réseau sans fil Wi-Fi :

   [...]
   Passerelle par défaut. . . . . . . . . : 10.33.19.254
```

### B - En graphique (GUI : Graphical User Interface)

#### - Trouvez comment afficher les informations sur une carte IP (change selon l'OS)

```bash
Sous Windows

- Aller dans le panneau de configuration
- Aller dans Réseau et Internet
- Cliquer sur la carte concernée
- Cliquer sur "Détails"
```

<div align="center">
<img src="./assets/adresseIPMAC.png" alt="Informations carte IP en graphique">
</div>

### Questions

```
La gateway du réseau d'Ynov nous sert à aller sur internet.
```

## 2. Modifications des informations

### A - Modification d'adresse IP (part 1)

```bash
Sous Windows

- Aller dans le panneau de configuration
- Aller dans Réseau et Internet
- Aller dans Modifier les paramètres de la page 
- Cliquer sur la carte concernée
- Cliquer sur "Propriétés"
- Cliquer sur "Protocole Internet Version 4 (TCP/IPv4)"
- Changer l'adresse IP
```

<div align="center">
<img src="./assets/changeadresseIP.png" alt="Changement adresse IP en graphique">
</div>


```bash
Il est possible de perdre l'accès à internet si l'adresse IP a déjà été selectionnée, la première personne a avoir choisi l'adresse IP sera prioritaire et la deuxième n'aura donc plus de réseau.
```
