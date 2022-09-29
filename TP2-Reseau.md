# TP2 : Ethernet, IP, et ARP

# O. Prérequis

```bash
Choix de PC + VM
```

# 1. Setup IP

### -  Mettez en place une configuration réseau fonctionnelle entre les deux machines


```bash
# changement d'adresse IP sur la VM
roxanne@roxanne-VirtualBox:/etc$ sudo nano netplan/01-network-manager-all.yaml
[sudo] password for roxanne:
roxanne@roxanne-VirtualBox:~$ cat /etc/netplan/01-network-manager-all.yaml
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    enp0s8:
      dhcp4: no
      addresses:
        - 192.168.121.4/26
      nameservers:
          addresses: [8.8.8.8, 1.1.1.1]


# adresse IP choisie pour la VM

192.168.121.4/26

# adresse IP choisie pour le PC

192.168.121.6/26

# le masque de sous-réseau

255.255.255.192
```

### - Prouvez que la connexion est fonctionnelle entre les deux machines

```bash
PS C:\Users\33785> ping 192.168.121.4

Envoi d’une requête 'Ping'  192.168.121.4 avec 32 octets de données :
Réponse de 192.168.121.4 : octets=32 temps<1ms TTL=64
Réponse de 192.168.121.4 : octets=32 temps=1 ms TTL=64
Réponse de 192.168.121.4 : octets=32 temps=1 ms TTL=64
Réponse de 192.168.121.4 : octets=32 temps<1ms TTL=64

Statistiques Ping pour 192.168.121.4:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 0ms, Maximum = 1ms, Moyenne = 0ms
```

### - Wireshark it

```bash
# les paquets ICMP envoyés sont des paquets de type 8 (request / ping) et de type 0 (reply / pong)
```