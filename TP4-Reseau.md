# TP4 : TCP, UDP et services réseau

Dans ce TP on va explorer un peu les protocoles TCP et UDP. On va aussi mettre en place des services qui font appel à ces protocoles.

# 0. Prérequis

```
Lu
```

# I. First steps

Faites-vous un petit top 5 des applications que vous utilisez sur votre PC souvent, des applications qui utilisent le réseau : un site que vous visitez souvent, un jeu en ligne, Spotify, j'sais po moi, n'importe.

🌞 **Déterminez, pour ces 5 applications, si c'est du TCP ou de l'UDP**

- avec Wireshark, on va faire les chirurgiens réseau
- déterminez, pour chaque application :
  - IP et port du serveur auquel vous vous connectez
  - le port local que vous ouvrez pour vous connecter


```bash
Site : Spotify
IP : 104.199.65.124
Src Port : 3311
Dst Port : 4070
Protocole : TCP
```

```bash
Site : Discord
IP : 162.159.133.234
Src Port : 11930
Dst Port : 443
Protocole : TCP
```

```bash
Site : Telegram
IP : 95.100.95.167
Src Port : 1110
Dst Port : 443
Protocole : TCP
```

```bash
Site : telemat.org
IP : 217.69.21.67
Src Port : 3681
Dst Port : 443
Protocole : TCP
```

```bash
Site : Star Stable
IP : 18.66.2.115
Src Port : 8540
Dst Port : 80
Protocole : TCP
```

> Dès qu'on se connecte à un serveur, notre PC ouvre un port random. Une fois la connexion TCP ou UDP établie, entre le port de notre PC et le port du serveur qui est en écoute, on parle de tunnel TCP ou de tunnel UDP.


> Aussi, TCP ou UDP ? Comment le client sait ? Il sait parce que le serveur a décidé ce qui était le mieux pour tel ou tel type de trafic (un jeu, une page web, etc.) et que le logiciel client est codé pour utiliser TCP ou UDP en conséquence.

🌞 **Demandez l'avis à votre OS**

- votre OS est responsable de l'ouverture des ports, et de placer un programme en "écoute" sur un port
- il est aussi responsable de l'ouverture d'un port quand une application demande à se connecter à distance vers un serveur
- bref il voit tout quoi
- utilisez la commande adaptée à votre OS pour repérer, dans la liste de toutes les connexions réseau établies, la connexion que vous voyez dans Wireshark, pour chacune des 5 applications

**Il faudra ajouter des options adaptées aux commandes pour y voir clair. Pour rappel, vous cherchez des connexions TCP ou UDP.**

```
# MacOS
$ netstat

# GNU/Linux
$ ss

# Windows
$ netstat
```

```bash
# Spotify
C:\Users\33785>netstat -an | findstr 4070
  TCP    192.168.1.20:3311      104.199.65.124:4070    ESTABLISHED
```
[Dump Spotify](./filesTP4/spotify.pcapng)

```bash
# Discord
C:\Users\33785>netstat -an | findstr 162.159
  TCP    192.168.1.20:1024      162.159.136.234:https  ESTABLISHED
```
[Dump Discord](./filesTP4/discorddeux.pcapng)

```bash
# Telegram
C:\Users\33785>netstat -an | findstr 1110
  TCP    192.168.1.20:1110      95.100.95.167:443      ESTABLISHED
```
[Dump Telegram](./filesTP4/telegram.pcapng)

```bash
# Telemat
C:\Users\33785>netstat -an | findstr 217.69.21.67
  TCP    192.168.1.20:3681      217.69.21.67:443       CLOSE_WAIT
  TCP    192.168.1.20:3684      217.69.21.67:443       CLOSE_WAIT

# les sessions HTTP sont terminées, les données sont envoyées, c'est pour ça que l'état est en CLOSE_WAIT
```
[Dump Telemat](./filesTP4/telemat.pcapng)

```bash
# Star Stable
C:\Users\33785>netstat -an | findstr 18.66.2.115
  TCP    192.168.1.20:8540      18.66.2.115:80         ESTABLISHED
```
[Dump Star Stable](./filesTP4/starstable.pcapng)

🦈🦈🦈🦈🦈 **Bah ouais, captures Wireshark à l'appui évidemment.** Une capture pour chaque application, qui met bien en évidence le trafic en question.

# II. Mise en place

Allumez une VM Linux pour la suite.


## 1. SSH

Connectez-vous en SSH à votre VM.

🌞 **Examinez le trafic dans Wireshark**

- donnez un sens aux infos devant vos yeux, capturez un peu de trafic, et coupez la capture, sélectionnez une trame random et regardez dedans, vous laissez pas brainfuck par Wireshark n_n
- **déterminez si SSH utilise TCP ou UDP**
  - pareil réfléchissez-y deux minutes, logique qu'on utilise pas UDP non ?
```bash
# on utilise le protocole TCP pour avoir confirmation de la reception des paquets
```
- **repérez le *3-Way Handshake* à l'établissement de la connexion**
  - c'est le `SYN` `SYNACK` `ACK`
- **repérez le FIN FINACK à la fin d'une connexion**
- entre le *3-way handshake* et l'échange `FIN`, c'est juste une bouillie de caca chiffré, dans un tunnel TCP

[Dump Syn/Synack/Ack + Fin](./filesTP4/synsynack.pcapng)

```bash
Malgré plusieurs essais, impossible d'obtenir un FINACK, la session se termine juste par un reset
```


🌞 **Demandez aux OS**

- repérez, avec un commande adaptée, la connexion SSH depuis votre machine
- ET repérez la connexion SSH depuis votre VM


```
# MacOS
$ netstat

# GNU/Linux
$ ss

# Windows
$ netstat
```

```bash
# depuis ma machine
C:\Users\33785>netstat -an | findstr 22
  TCP    10.3.1.11:1633         10.3.1.13:22           ESTABLISHED
```

```bash
# depuis ma VM
[roxanne@localhost ~]$ netstat -an | grep 22
[...]
tcp        0     52 10.3.1.13:22            10.3.1.11:1633          ESTABLISHED
```

🦈 **Je veux une capture clean avec le 3-way handshake, un peu de trafic au milieu et une fin de connexion**

## 2. NFS

Allumez une deuxième VM Linux pour cette partie.

Vous allez installer un serveur NFS. Un serveur NFS c'est juste un programme qui écoute sur un port (comme toujours en fait, oèoèoè) et qui propose aux clients d'accéder à des dossiers à travers le réseau.

Une de vos VMs portera donc le serveur NFS, et l'autre utilisera un dossier à travers le réseau.

🌞 **Mettez en place un petit serveur NFS sur l'une des deux VMs**

- j'vais pas ré-écrire la roue, google it, ou [go ici](https://www.server-world.info/en/note?os=Rocky_Linux_8&p=nfs&f=1)
- partagez un dossier que vous avez créé au préalable dans `/srv`
- vérifiez que vous accédez à ce dossier avec l'autre machine : [le client NFS](https://www.server-world.info/en/note?os=Rocky_Linux_8&p=nfs&f=2)



> Si besoin, comme d'hab, je peux aider à la compréhension, n'hésitez pas à m'appeler.

🌞 **Wireshark it !**

- une fois que c'est en place, utilisez `tcpdump` pour capturer du trafic NFS
- déterminez le port utilisé par le serveur

🌞 **Demandez aux OS**

- repérez, avec un commande adaptée, la connexion NFS sur le client et sur le serveur

```
# GNU/Linux
$ ss
```

🦈 **Et vous me remettez une capture de trafic NFS** la plus complète possible. J'ai pas dit que je voulais le plus de trames possible, mais juste, ce qu'il faut pour avoir un max d'infos sur le trafic

## 3. DNS

🌞 Utilisez une commande pour effectuer une requête DNS depuis une des VMs

- capturez le trafic avec un `tcpdump`
- déterminez le port et l'IP du serveur DNS auquel vous vous connectez
