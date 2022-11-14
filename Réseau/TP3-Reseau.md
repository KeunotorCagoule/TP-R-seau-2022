# TP3 : On va router des trucs

Au menu de ce TP, on va revoir un peu ARP et IP histoire de **se mettre en jambes dans un environnement avec des VMs**.

Puis on mettra en place **un routage simple, pour permettre à deux LANs de communiquer**.


## Sommaire

- [TP3 : On va router des trucs](#tp3--on-va-router-des-trucs)
  - [Sommaire](#sommaire)
  - [0. Prérequis](#0-prérequis)
  - [I. ARP](#i-arp)
    - [1. Echange ARP](#1-echange-arp)
    - [2. Analyse de trames](#2-analyse-de-trames)
  - [II. Routage](#ii-routage)
    - [1. Mise en place du routage](#1-mise-en-place-du-routage)
    - [2. Analyse de trames](#2-analyse-de-trames-1)
    - [3. Accès internet](#3-accès-internet)
  - [III. DHCP](#iii-dhcp)
    - [1. Mise en place du serveur DHCP](#1-mise-en-place-du-serveur-dhcp)
    - [2. Analyse de trames](#2-analyse-de-trames-2)

## 0. Prérequis

Fait.

## I. ARP

Fait.

```bash
# ayant eu des petits soucis de clonage et d'installation, les ip des vm sont 10.1.3.13 et 10.1.3.14
```

### 1. Echange ARP

🌞**Générer des requêtes ARP**

- effectuer un `ping` d'une machine à l'autre
```
[root@localhost ~]# ping 10.3.1.14
PING 10.3.1.14 (10.3.1.14) 56(84) bytes of data.
64 bytes from 10.3.1.14: icmp_seq=1 ttl=64 time=0.747 ms
64 bytes from 10.3.1.14: icmp_seq=2 ttl=64 time=0.418 ms
64 bytes from 10.3.1.14: icmp_seq=3 ttl=64 time=0.331 ms
64 bytes from 10.3.1.14: icmp_seq=4 ttl=64 time=0.363 ms
^C
--- 10.3.1.14 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3052ms
rtt min/avg/max/mdev = 0.331/0.464/0.747/0.165 ms
```

```
[root@localhost ~]# ping 10.3.1.13
PING 10.3.1.13 (10.3.1.13) 56(84) bytes of data.
64 bytes from 10.3.1.13: icmp_seq=1 ttl=64 time=0.540 ms
64 bytes from 10.3.1.13: icmp_seq=2 ttl=64 time=0.526 ms
64 bytes from 10.3.1.13: icmp_seq=3 ttl=64 time=0.398 ms
64 bytes from 10.3.1.13: icmp_seq=4 ttl=64 time=0.521 ms
^C
--- 10.3.1.13 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3112ms
rtt min/avg/max/mdev = 0.398/0.496/0.540/0.057 ms
```
- observer les tables ARP des deux machines
- repérer l'adresse MAC de `john` dans la table ARP de `marcel` et vice-versa
```
// adresse MAC de john dans la table ARP de marcel
[root@localhost ~]# ip neigh show 10.3.1.13
[...] 08:00:27:a0:14:1b [...]
```

```
// adresse MAC de marcel dans la table ARP de john
[root@localhost ~]# ip neigh show 10.3.1.14
[...]08:00:27:5f:29:9e[...]
```
- prouvez que l'info est correcte (que l'adresse MAC que vous voyez dans la table est bien celle de la machine correspondante)
  - une commande pour voir la MAC de `marcel` dans la table ARP de `john`
  - et une commande pour afficher la MAC de `marcel`, depuis `marcel`
```
[root@localhost ~]# ip neigh show 10.3.1.13
10.3.1.13 dev enp0s8 lladdr 08:00:27:a0:14:1b STALE

[root@localhost ~]$ ip a
[...]
2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:a0:14:1b brd ff:ff:ff:ff:ff:ff
[...]
```

### 2. Analyse de trames

🌞**Analyse de trames**

- utilisez la commande `tcpdump` pour réaliser une capture de trame
- videz vos tables ARP, sur les deux machines, puis effectuez un `ping`

```
[root@localhost ~]# tcpdump arp -w ping.pcap
dropped privs to tcpdump
tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), snapshot length 262144 bytes
^C4 packets captured
4 packets received by filter
0 packets dropped by kernel


[root@localhost ~]# ip n flush all
[root@localhost ~]# ping 10.3.1.13
PING 10.3.1.13 (10.3.1.13) 56(84) bytes of data.
64 bytes from 10.3.1.13: icmp_seq=1 ttl=64 time=0.573 ms
64 bytes from 10.3.1.13: icmp_seq=2 ttl=64 time=0.544 ms
64 bytes from 10.3.1.13: icmp_seq=3 ttl=64 time=0.452 ms
^C
--- 10.3.1.13 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2042ms
rtt min/avg/max/mdev = 0.452/0.523/0.573/0.051 ms


```

🦈 [**Capture réseau `tp2_arp.pcapng`** qui contient un ARP request et un ARP reply](./filesTP2/ping.pcapng)

> **Si vous ne savez pas comment récupérer votre fichier `.pcapng`** sur votre hôte afin de l'ouvrir dans Wireshark, et me le livrer en rendu, demandez-moi.

## II. Routage

Vous aurez besoin de 3 VMs pour cette partie. **Réutilisez les deux VMs précédentes.**

| Machine  | `10.3.1.0/24` | `10.3.2.0/24` |
|----------|---------------|---------------|
| `router` | `10.3.1.254`  | `10.3.2.254`  |
| `john`   | `10.3.1.11`   | no            |
| `marcel` | no            | `10.3.2.12`   |

> Je les appelés `marcel` et `john` PASKON EN A MAR des noms nuls en réseau 🌻

```schema
   john                router              marcel
  ┌─────┐             ┌─────┐             ┌─────┐
  │     │    ┌───┐    │     │    ┌───┐    │     │
  │     ├────┤ho1├────┤     ├────┤ho2├────┤     │
  └─────┘    └───┘    └─────┘    └───┘    └─────┘
```

### 1. Mise en place du routage

🌞**Activer le routage sur le noeud `router`**

> Cette étape est nécessaire car Rocky Linux c'est pas un OS dédié au routage par défaut. Ce n'est bien évidemment une opération qui n'est pas nécessaire sur un équipement routeur dédié comme du matériel Cisco.

```
[root@localhost ~]# echo 1 > /proc/sys/net/ipv4/ip_forward
```

🌞**Ajouter les routes statiques nécessaires pour que `john` et `marcel` puissent se `ping`**

- il faut ajouter une seule route des deux côtés
- une fois les routes en place, vérifiez avec un `ping` que les deux machines peuvent se joindre

```
[root@localhost ~]# route add default gw 10.3.1.254
[root@localhost ~]# netstat -rn
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         10.3.1.254      0.0.0.0         UG        0 0          0 enp0s8
10.3.1.0        0.0.0.0         255.255.255.0   U         0 0          0 enp0s8
```

```
[root@localhost ~]# route add default gw 10.3.2.254
[root@localhost ~]# netstat -rn
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         10.3.2.254      0.0.0.0         UG        0 0          0 enp0s8
10.3.2.0        0.0.0.0         255.255.255.0   U         0 0          0 enp0s8
```

```
[root@localhost ~]# ping 10.3.1.13
PING 10.3.1.13 (10.3.1.13) 56(84) bytes of data.
64 bytes from 10.3.1.13: icmp_seq=1 ttl=63 time=1.32 ms
64 bytes from 10.3.1.13: icmp_seq=2 ttl=63 time=1.30 ms
64 bytes from 10.3.1.13: icmp_seq=3 ttl=63 time=1.04 ms
^C
--- 10.3.1.13 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 1.040/1.218/1.320/0.126 ms



[root@localhost ~]# ping 10.3.2.12
PING 10.3.2.12 (10.3.2.12) 56(84) bytes of data.
64 bytes from 10.3.2.12: icmp_seq=1 ttl=63 time=1.27 ms
64 bytes from 10.3.2.12: icmp_seq=2 ttl=63 time=1.19 ms
64 bytes from 10.3.2.12: icmp_seq=3 ttl=63 time=1.12 ms
^C
--- 10.3.2.12 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2005ms
rtt min/avg/max/mdev = 1.123/1.193/1.271/0.060 ms
```

### 2. Analyse de trames

🌞**Analyse des échanges ARP**

- videz les tables ARP des trois noeuds
- effectuez un `ping` de `john` vers `marcel`
- regardez les tables ARP des trois noeuds
- essayez de déduire un peu les échanges ARP qui ont eu lieu
- répétez l'opération précédente (vider les tables, puis `ping`), en lançant `tcpdump` sur `marcel`
- **écrivez, dans l'ordre, les échanges ARP qui ont eu lieu, puis le ping et le pong, je veux TOUTES les trames** utiles pour l'échange

Par exemple (copiez-collez ce tableau ce sera le plus simple) :

| ordre | type trame  | IP source | MAC source              | IP destination | MAC destination            |
|-------|-------------|-----------|-------------------------|----------------|----------------------------|
| 1     | Requête ARP | x         | `john` `AA:BB:CC:DD:EE` | x              | Broadcast `FF:FF:FF:FF:FF` |
| 2     | Réponse ARP | x         | ?                       | x              | `john` `AA:BB:CC:DD:EE`    |
| ...   | ...         | ...       | ...                     |                |                            |
| ?     | Ping        | ?         | ?                       | ?              | ?                          |
| ?     | Pong        | ?         | ?                       | ?              | ?                          |

> Vous pourriez, par curiosité, lancer la capture sur `john` aussi, pour voir l'échange qu'il a effectué de son côté.

🦈 [**Capture réseau `tp2_routage_marcel.pcapng`**](./filesTP2/arp.pcapng)

### 3. Accès internet

🌞**Donnez un accès internet à vos machines**

- ajoutez une carte NAT en 3ème inteface sur le `router` pour qu'il ait un accès internet
- ajoutez une route par défaut à `john` et `marcel`
  - vérifiez que vous avez accès internet avec un `ping`
  - le `ping` doit être vers une IP, PAS un nom de domaine
- donnez leur aussi l'adresse d'un serveur DNS qu'ils peuvent utiliser
  - vérifiez que vous avez une résolution de noms qui fonctionne avec `dig`
  - puis avec un `ping` vers un nom de domaine

🌞**Analyse de trames**

- effectuez un `ping 8.8.8.8` depuis `john`
- capturez le ping depuis `john` avec `tcpdump`
- analysez un ping aller et le retour qui correspond et mettez dans un tableau :

| ordre | type trame | IP source          | MAC source              | IP destination | MAC destination |     |
|-------|------------|--------------------|-------------------------|----------------|-----------------|-----|
| 1     | ping       | `john` `10.3.1.12` | `john` `AA:BB:CC:DD:EE` | `8.8.8.8`      | ?               |     |
| 2     | pong       | ...                | ...                     | ...            | ...             | ... |

🦈 **Capture réseau `tp2_routage_internet.pcapng`**

## III. DHCP

On reprend la config précédente, et on ajoutera à la fin de cette partie une 4ème machine pour effectuer des tests.

| Machine  | `10.3.1.0/24`              | `10.3.2.0/24` |
|----------|----------------------------|---------------|
| `router` | `10.3.1.254`               | `10.3.2.254`  |
| `john`   | `10.3.1.11`                | no            |
| `bob`    | oui mais pas d'IP statique | no            |
| `marcel` | no                         | `10.3.2.12`   |

```schema
   john               router              marcel
  ┌─────┐             ┌─────┐             ┌─────┐
  │     │    ┌───┐    │     │    ┌───┐    │     │
  │     ├────┤ho1├────┤     ├────┤ho2├────┤     │
  └─────┘    └─┬─┘    └─────┘    └───┘    └─────┘
   john        │
  ┌─────┐      │
  │     │      │
  │     ├──────┘
  └─────┘
```

### 1. Mise en place du serveur DHCP

🌞**Sur la machine `john`, vous installerez et configurerez un serveur DHCP** (go Google "rocky linux dhcp server").

- installation du serveur sur `john`
- créer une machine `bob`
- faites lui récupérer une IP en DHCP à l'aide de votre serveur

> Il est possible d'utilise la commande `dhclient` pour forcer à la main, depuis la ligne de commande, la demande d'une IP en DHCP, ou renouveler complètement l'échange DHCP (voir `dhclient -h` puis call me et/ou Google si besoin d'aide).

🌞**Améliorer la configuration du DHCP**

- ajoutez de la configuration à votre DHCP pour qu'il donne aux clients, en plus de leur IP :
  - une route par défaut
  - un serveur DNS à utiliser
- récupérez de nouveau une IP en DHCP sur `bob` pour tester :
  - `marcel` doit avoir une IP
    - vérifier avec une commande qu'il a récupéré son IP
    - vérifier qu'il peut `ping` sa passerelle
  - il doit avoir une route par défaut
    - vérifier la présence de la route avec une commande
    - vérifier que la route fonctionne avec un `ping` vers une IP
  - il doit connaître l'adresse d'un serveur DNS pour avoir de la résolution de noms
    - vérifier avec la commande `dig` que ça fonctionne
    - vérifier un `ping` vers un nom de domaine

### 2. Analyse de trames

🌞**Analyse de trames**

- lancer une capture à l'aide de `tcpdump` afin de capturer un échange DHCP
- demander une nouvelle IP afin de générer un échange DHCP
- exportez le fichier `.pcapng`

🦈 **Capture réseau `tp2_dhcp.pcapng`**