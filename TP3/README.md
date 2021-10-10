# **TP3 : Progressons vers le réseau d'infrastructure**

# **Sommaire**

- [TP3 : Progressons vers le réseau d'infrastructure](#tp3--progressons-vers-le-réseau-dinfrastructure)
- [Sommaire](#sommaire)
- [I. (mini)Architecture réseau](#i-miniarchitecture-réseau)
  - [1. Adressage](#1-adressage)
  - [2. Routeur](#2-routeur)
- [II. Services d'infra](#ii-services-dinfra)
  - [1. Serveur DHCP](#1-serveur-dhcp)
  - [2. Serveur DNS](#2-serveur-dns)
    - [A. Our own DNS server](#a-our-own-dns-server)
    - [B. SETUP copain](#b-setup-copain)
  - [3. Get deeper](#3-get-deeper)
    - [A. DNS forwarder](#a-dns-forwarder)
    - [B. On revient sur la conf du DHCP](#b-on-revient-sur-la-conf-du-dhcp)
- [Entracte](#entracte)
- [III. Services métier](#iii-services-métier)
  - [1. Serveur Web](#1-serveur-web)
  - [2. Partage de fichiers](#2-partage-de-fichiers)
    - [A. L'introduction wola](#a-lintroduction-wola)
    - [B. Le setup wola](#b-le-setup-wola)
- [IV. Un peu de théorie : TCP et UDP](#iv-un-peu-de-théorie-tcp-et-udp)
- [V. El final](#v-el-final)

# **I. (mini)Architecture réseau**
## **1. Adressage**


## **A lire important**

> **Pour les tableaux des réseaux et d'adressage, tout était fonctionnel mais je n'ai pas utilisé les IP entre 10.3.0.64 et 10.3.0.127.**

> **J'étais déjà à la fin du TP, ducoup ici j'ai juste mis ceux que j'ai utilisé pour le TP et dans le [V. Bilan](#v-el-final), j'ai mis comme tu m'avais demandé les tableaux que j'ai utilisés et ceux que j'aurais du utiliser avec des IP qui se suivent et qui sont toutes dans le réseau 10.3.0.0/24**

#### **🌞 Vous me rendrez un 🗃️ tableau des réseaux 🗃️ qui rend compte des adresses choisies, sous la forme** :

> **Tableau des réseaux que j'ai utilisé pour le TP**

| Nom du réseau | Adresse du réseau | Masque            | Nombre de clients possibles | Adresse passerelle | Adresse broadcast |
|---------------|-------------------|-------------------|-----------------------------|--------------------|-----------------|
| `client1`     | `10.3.0.0`        | `255.255.255.192` | 61                          | `10.3.0.62`        | `10.3.0.63`       |
| `server1`     | `10.3.0.128`      | `255.255.255.128` | 125                         | `10.3.0.254`       | `10.3.0.255`      |
| `server2`     | `10.3.1.0`        | `255.255.255.240` | 13                          | `10.3.1.14`        | `10.3.1.15`       |

#### **🌞 Vous remplirez aussi au fur et à mesure que vous avancez dans le TP, au fur et à mesure que vous créez des machines, le 🗃️ tableau d'adressage 🗃️ suivant :**

> **Tableau d'adressage que j'ai utilisé pour le TP**
        
| Nom machine           | Adresse IP `client1` | Adresse IP `server1` | Adresse IP `server2` | Adresse de passerelle |
| --------------------- | -------------------- | -------------------- | -------------------- | --------------------- |
| `router.tp3`          | `10.3.0.62/26`       | `10.3.0.254/25`      | `10.3.1.14/28`       | Carte NAT             |
| `dhcp.client1.tp3`    | `10.3.0.61/26`       | ...                  | ...                  | `10.3.0.62/26`        |
| `marcel.client1.tp3`  | `10.3.0.11/26`       | ...                  | ...                  | `10.3.0.62/26`        |
| `johnny.client1.tp3`  | `10.3.0.13/26`       | ...                  | ...                  | `10.3.0.62/26`        |
| `dns1.server1.tp3`    | ...                  | `10.3.0.253/25`      | ...                  | `10.3.0.254/25`       |
| `web1.server2.tp3`    | ...                  | ...                  | `10.3.1.2/28`        | `10.3.1.14/28 `       |
| `nfs1.server2.tp3`    | ...                  | ...                  | `10.3.1.3/28`        | `10.3.1.14/28 `       |

## **2. Routeur**

#### **🌞 Vous pouvez d'ores-et-déjà créer le routeur. Pour celui-ci, vous me prouverez que :**
- **Il a bien une IP dans les 3 réseaux, l'IP que vous avez choisie comme IP de passerelle**
- **Il a un accès internet :**

| Nom du réseau : | NAT          | 10.3.0.0/26 | 10.3.0.128/25 | 10.3.1.0/28 |
| --------------- | -------------| ----------- | ------------- | ----------- |
| IP du routeur : | 10.0.2.15/24 | 10.3.0.62   | 10.3.0.254    | 10.3.1.14   |

```
[yrlan@router ~]$ ip r s
default via 10.0.2.2 dev enp0s3 proto dhcp metric 101
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 101
10.3.0.0/26 dev enp0s8 proto kernel scope link src 10.3.0.62 metric 102
10.3.0.128/25 dev enp0s9 proto kernel scope link src 10.3.0.254 metric 103
10.3.1.0/28 dev enp0s10 proto kernel scope link src 10.3.1.14 metric 100

[yrlan@router ~]$ ip a
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:a6:43:12 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
       valid_lft 75494sec preferred_lft 75494sec
    inet6 fe80::a00:27ff:fea6:4312/64 scope link noprefixroute
   valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:f5:a8:76 brd ff:ff:ff:ff:ff:ff
    inet 10.3.0.62/26 brd 10.3.0.63 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fef5:a876/64 scope link
       valid_lft forever preferred_lft forever
4: enp0s9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:bc:fc:3d brd ff:ff:ff:ff:ff:ff
    inet 10.3.0.254/25 brd 10.3.0.255 scope global noprefixroute enp0s9
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:febc:fc3d/64 scope link
       valid_lft forever preferred_lft forever
5: enp0s10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:3f:30:85 brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.14/28 brd 10.3.1.15 scope global noprefixroute enp0s10
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe3f:3085/64 scope link
       valid_lft forever preferred_lft forever

[yrlan@router ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=114 time=25.6 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=114 time=20.9 ms
^C
--- 8.8.8.8 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 20.935/23.279/25.624/2.349 ms
```
    
- **Il a de la résolution de noms :**
    ```
    [yrlan@router ~]$ dig ynov.com
    [...]
    ;; ANSWER SECTION:
    ynov.com.               2531    IN      A       92.243.16.143

    ;; Query time: 21 msec
    ;; SERVER: 1.1.1.1#53(1.1.1.1)
    ;; WHEN: Mon Sep 27 17:28:10 CEST 2021
    ;; MSG SIZE  rcvd: 53
    ```
    `ynov.com.               2531    IN      A       92.243.16.143`
- **Il porte le nom `router.tp3`**
    ```
    [yrlan@dhcp ~]$ echo 'router.tp3' | sudo tee /etc/hostname
    router.tp3
    [yrlan@router ~]$ hostname
    router.tp3
    ```
    
- **N'oubliez pas d'activer le routage sur la machine**
    ```
    [yrlan@router ~]$ sudo firewall-cmd --add-masquerade --zone=public --permanent
    success
    [yrlan@router ~]$ sudo firewall-cmd --reload
    success
    [yrlan@router ~]$ sudo firewall-cmd --list-all
    public (active)
      target: default
      icmp-block-inversion: no
      interfaces: enp0s10 enp0s3 enp0s8 enp0s9
      sources:
      services: ssh
      ports:
      protocols:
      masquerade: yes
      -ports:
      source-ports:
      icmp-blocks:
      rich rules:
    ```


# **II. Services d'infra**
## **1. Serveur DHCP**

**🖥️ VM `dhcp.client1.tp3`**
#### **🌞 Mettre en place une machine qui fera office de serveur DHCP dans le réseau `client1`. Elle devra :**

- **Porter le nom `dhcp.client1.tp3` :**
    ```
    [yrlan@dhcp ~]$ echo 'dhcp.client1.tp3' | sudo tee /etc/hostname
    dhcp.client1.tp3
    [yrlan@dhcp ~]$ hostname
    dhcp.client1.tp3
    ```
    
- **Donner une IP aux machines clients qui le demandent :**
    - **Leur donner l'adresse de leur passerelle : `option routers`**
    - **Leur donner l'adresse d'un DNS utilisable : `option domain-name-servers`**
    ```
    [yrlan@dhcp ~]$ sudo dnf -y install dhcp-server
    [yrlan@dhcp ~]$ sudo firewall-cmd --zone=public --permanent --add-service=dhcp; sudo firewall-cmd --reload; sudo firewall-cmd --list-all
    success
    success
    public (active)
      target: default
      icmp-block-inversion: no
      interfaces: enp0s8
      sources:
      services: dhcp ssh
      ports:
      protocols:
      masquerade: no
      forward-ports:
      source-ports:
      icmp-blocks:
      rich rules:

    [yrlan@dhcp ~]$ sudo cat /etc/dhcp/dhcpd.conf
    #
    # DHCP Server Configuration file.
    #   see /usr/share/doc/dhcp-server/dhcpd.conf.example
    #   see dhcpd.conf(5) man page
    #
    # Bail de 24H, max 48H
    default-lease-time 86400;
    max-lease-time 172800;

    # Déclaration du réseau "client1" (10.3.0.0/26) + range
    subnet 10.3.0.0 netmask 255.255.255.192 {
            range                           10.3.0.10 10.3.0.50;
            option domain-name-servers      1.1.1.1;
            option routers                  10.3.0.62;
    }
    
    [yrlan@dhcp ~]$ sudo systemctl start dhcpd; sudo systemctl enable dhcpd
    Created symlink /etc/systemd/system/multi-user.target.wants/dhcpd.service → /usr/lib/systemd/system/dhcpd.service.
    ```
> **📁 [Fichier de conf du serveur DHCP : `dhcpd.conf`](./conf/dhcpd.conf)**

> **Le `option domain-name-servers` dans le fichier au dessus est `10.3.0.253` qui est l'IP de mon serveur DNS ( modifié après pendant le TP pour donner automatiquement le DNS)**

#### **🌞 Mettre en place un client dans le réseau `client1` :**
> **🖥️ VM `marcel.client1.tp3`**

- **De son p'tit nom `marcel.client1.tp3`**
    - **La machine récupérera une IP dynamiquement grâce au serveur DHCP**
    - **Ainsi que sa passerelle et une adresse d'un DNS utilisable**

   ```
    [yrlan@marcel ~]$ cat /etc/sysconfig/network-scripts/ifcfg-enp0s8
    TYPE=Ethernet
    BOOTPROTO=dhcp
    DEFROUTE=yes
    NAME=enp0s8
    UUID=c077c523-fb65-4a2a-8998-c40b1466e030
    DEVICE=enp0s8
    ONBOOT=yes

    [yrlan@marcel ~]$ sudo dhclient
    [yrlan@marcel ~]$ sudo nmcli device show enp0s8
    [...]
    IP4.ADDRESS[1]:                         10.3.0.11/26
    IP4.GATEWAY:                            10.3.0.62
    IP4.DNS[1]:                             1.1.1.1
    [...]
    ```
    
#### **🌞 Depuis `marcel.client1.tp3` :**
- **Prouver qu'il a un accès internet + résolution de noms, avec des infos récupérées par votre DHCP**
    ```
    [yrlan@marcel ~]$ sudo dhclient -r; sudo dhclient -v
    Killed old client process
    [...]
    DHCPDISCOVER on enp0s8 to 255.255.255.255 port 67 interval 8 (xid=0x2379bb5a)
    DHCPREQUEST on enp0s8 to 255.255.255.255 port 67 (xid=0x2379bb5a)
    DHCPOFFER from 10.3.0.61
    DHCPACK from 10.3.0.61 (xid=0x2379bb5a)
    bound to 10.3.0.11 -- renewal in 42656 seconds.
    
    [yrlan@dhcp ~]$ sudo cat /var/lib/dhcpd/dhcpd.leases
    [...]
    lease 10.3.0.11 {
      starts 2 2021/09/28 00:01:51;
      ends 3 2021/09/29 00:01:51;
      cltt 2 2021/09/28 00:01:51;
      binding state active;
      next binding state free;
      rewind binding state free;
      hardware ethernet 08:00:27:46:38:22;
      uid "\001\010\000'F8\"";
      client-hostname "marcel";
    }
    [...]
    
    ## Route par défaut = router.tp3
    [yrlan@marcel ~]$ ip r s
    default via 10.3.0.62 dev enp0s8 proto dhcp metric 100
    10.3.0.0/26 dev enp0s8 proto kernel scope link src 10.3.0.11 metric 100

    [yrlan@marcel ~]$ ping 8.8.8.8
    PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
    64 bytes from 8.8.8.8: icmp_seq=1 ttl=112 time=13.2 ms
    
    --- 8.8.8.8 ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 13.203/13.203/13.203/0.000 ms
    
    [yrlan@marcel ~]$ dig google.com
    ; <<>> DiG 9.11.26-RedHat-9.11.26-4.el8_4 <<>> google.com
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 50717
    ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 1232
    ;; QUESTION SECTION:
    ;google.com.                    IN      A

    ;; ANSWER SECTION:
    google.com.             87      IN      A       216.58.209.238

    ;; Query time: 20 msec
    ;; SERVER: 1.1.1.1#53(1.1.1.1)
    ;; WHEN: Tue Sep 28 02:10:32 CEST 2021
    ;; MSG SIZE  rcvd: 55
    ```
    
- **A l'aide de la commande traceroute, prouver que `marcel.client1.tp3` passe par `router.tp3` pour sortir de son réseau**
    ```
    [yrlan@marcel ~]$ traceroute -4 8.8.8.8
    traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
     1  _gateway (10.3.0.62)  1.764 ms  1.730 ms  1.643 ms
     2  10.0.2.2 (10.0.2.2)  1.617 ms  1.469 ms  1.434 ms
     3  10.0.2.2 (10.0.2.2)  2.399 ms  2.232 ms  1.906 ms
    ```
    > **On voit (ligne1) que `marcel.client1.tp3` passe par `router.tp3` (10.3.0.62) pour sortir de son réseau**

## **2. Serveur DNS**

### **A. Our own DNS server**
**A la fin donc, votre serveur sera :**

- **Autoritatif pour les zones `server1.tp3` et `server2.tp3`**
- **Resolver local**
- **Forwarder (pour résoudre les noms publics)**

### **B. SETUP copain**

> **🖥️ VM `dns1.server1.tp3`**

#### **🌞 Mettre en place une machine qui fera office de serveur DNS**
- **Dans le réseau `server1 ( 10.3.0.128/25 )`**
    - **De son p'tit nom `dns1.server1.tp3` :**
    ```
    [yrlan@dns1 ~]$ sudo cat /etc/sysconfig/network-scripts/ifcfg-enp0s8
    TYPE=Ethernet
    BOOTPROTO=static
    IPADDR=10.3.0.253
    NETMASK=255.255.255.128
    DEFROUTE=yes
    NAME=enp0s8
    UUID=c077c523-fb65-4a2a-8998-c40b1466e030
    DEVICE=enp0s8
    ONBOOT=yes
    GATEWAY=10.3.0.254
    DNS1=10.3.0.253
    
    [yrlan@dns1 ~]$ sudo nmcli device show enp0s8
    [...]
    IP4.ADDRESS[1]:                         10.3.0.253/25
    IP4.GATEWAY:                            10.3.0.254
    IP4.DNS[1]:                             10.3.0.253
    [...]
    
    [yrlan@dns1 ~]$ hostname
    dns1.server1.tp3
    ```

- **Il faudra lui ajouter un serveur DNS public connu, afin qu'il soit capable de résoudre des noms publics comme google.com :**
    ```
    [yrlan@dns1 ~]$ sudo cat /etc/resolv.conf
    # Generated by NetworkManager
    nameserver 1.1.1.1

    [yrlan@dns1 ~]$ dig google.com

    ; <<>> DiG 9.11.26-RedHat-9.11.26-4.el8_4 <<>> google.com
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 49706
    ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 1232
    ;; QUESTION SECTION:
    ;google.com.                    IN      A

    ;; ANSWER SECTION:
    google.com.             198     IN      A       172.217.22.142

    ;; Query time: 19 msec
    ;; SERVER: 1.1.1.1#53(1.1.1.1)
    ;; WHEN: Tue Sep 28 03:12:45 CEST 2021
    ;; MSG SIZE  rcvd: 55
    ```
    > Résolution du nom google.com : 
    `google.com.             198     IN      A       172.217.22.142`

- **Comme pour le DHCP, on part sur "rocky linux dns server" on Google pour les détails de l'install**   
    - **Le paquet que vous allez installer devrait s'appeler bind : c'est le nom du serveur DNS le plus utilisé au monde**
    ```
    [yrlan@dns1 ~]$ sudo dnf install -y bind bind-utils
    [yrlan@dns1 ~]$ sudo firewall-cmd --add-service=dns --permanent; sudo firewall-cmd --reload; sudo firewall-cmd --list-all
    success
    success
    public (active)
      target: default
      icmp-block-inversion: no
      interfaces: enp0s8
      sources:
      services: dns ssh
      ports:
      protocols:
      masquerade: no
      forward-ports:
      source-ports:
      icmp-blocks:
      rich rules:

    [yrlan@dns1 ~]$ sudo nano /etc/named.conf
    [yrlan@dns1 ~]$ sudo nano /var/named/client1.tp3.forward
    [yrlan@dns1 ~]$ sudo nano /var/named/server1.tp3.forward
    [yrlan@dns1 ~]$ sudo nano /var/named/server2.tp3.forward
    [yrlan@dns1 ~]$ sudo systemctl enable named; sudo systemctl start named
    ```

- **Il y aura plusieurs fichiers de conf :**
    - **Un fichier de conf principal `named.conf`**
    ```
    // named.conf
    
    acl "allowed" {
            10.3.0.0/24;
            10.3.0.1/28;
    };

    options {
            listen-on port 53 { 127.0.0.1; 10.3.0.253; };
            listen-on-v6 port 53 { ::1; };

            directory       "/var/named";
            dump-file       "/var/named/data/cache_dump.db";
            statistics-file "/var/named/data/named_stats.txt";
            memstatistics-file "/var/named/data/named_mem_stats.txt";
            secroots-file   "/var/named/data/named.secroots";
            recursing-file  "/var/named/data/named.recursing";

            allow-query     { localhost; allowed; };
            recursion       yes;
            dnssec-enable   yes;
            dnssec-validation yes;

            managed-keys-directory "/var/named/dynamic";
            pid-file "/run/named/named.pid";
            session-keyfile "/run/named/session.key";
            include "/etc/crypto-policies/back-ends/bind.config";
    };

    logging {
            channel default_debug {
                    file "data/named.run";
                    severity dynamic;
            };
    };

    # Déclaration des zones
    zone "client1.tp3" IN {
            type master;
            file "/var/named/client1.tp3.forward";
            allow-update { none; };
    };

    zone "server1.tp3" IN {
            type master;
            file "/var/named/server1.tp3.forward";
            allow-update { none; };
    };

    zone "server2.tp3" IN {
            type master;
            file "/var/named/server2.tp3.forward";
            allow-update { none; };
    };

    include "/etc/named.rfc1912.zones";
    include "/etc/named.root.key";
    ```
    > Pour vérifier la conf de `named.conf`, la commande ne doit rien retourner s'il n'y a pas d'erreur de syntaxe
    ```
    [yrlan@dns1 ~]$ sudo named-checkconf
    ```

    - **Des fichiers de zone "forward"**
        - **`client1.tp3.forward` (juste pour dhcp qui a une IP fixe)**
        ```
        [yrlan@dns1 ~]$ sudo cat /var/named/client1.tp3.forward
        [sudo] password for yrlan:
        $TTL 86400
        @       IN SOA  dns1.server1.tp3. root.server1.tp3 (
                        2021062301   ; serial
                        21600        ; refresh
                        3600         ; retry
                        604800       ; expire
                        86400 )      ; minimum TTL

        ; Define nameservers
        @       IN NS dns1.server1.tp3.
        ; Clients record
        dhcp    IN A 10.3.0.61
        ```
        > Vérifier la conf de la zone `client1.tp3`
        ```
        [yrlan@dns1 ~]$ sudo named-checkzone client1.tp3 /var/named/client1.tp3.forward
        zone client1.tp3/IN: loaded serial 2021062301
        OK
        ```

        - **`server1.tp3.forward`**
        ```
        [yrlan@dns1 ~]$ sudo cat /var/named/server1.tp3.forward
        $TTL 86400
        @       IN SOA  dns1.server1.tp3. root.server1.tp3 (
                        2021062301   ; serial
                        21600        ; refresh
                        3600         ; retry
                        604800       ; expire
                        86400 )      ; minimum TTL

        ; Define nameservers
        @       IN NS   dns1.server1.tp3.
        ; DNS Server IP Adresses
        dns1    IN A    10.3.0.253
        ```
        > Vérifier la conf de la zone `server1.tp3`
        ```
        [yrlan@dns1 ~]$ sudo named-checkzone server1.tp3 /var/named/server1.tp3.forward
        zone server1.tp3/IN: loaded serial 2021062301
        OK
        ```

        - **`server2.tp3.forward` (j'ai rajouté web1 et nfs1 après comme je n'avais pas encore de machines dans server2)**
        ```
        $TTL 86400
        @       IN SOA  dns1.server1.tp3. root.server1.tp3 (
                        2021062301   ; serial
                        21600        ; refresh
                        3600         ; retry
                        604800       ; expire
                        86400 )      ; minimum TTL

        ; Define nameservers
        @       IN NS   dns1.server1.tp3.
        ; Clients records
        web1    IN A    10.3.1.2
        nfs1    IN A    10.3.1.3
        ```
        > Vérifier la conf de la zone `server2.tp3`
        ```
        [yrlan@dns1 ~]$ sudo named-checkzone server2.tp3 /var/named/server2.tp3.forward
        zone server2.tp3/IN: loaded serial 2021062301
        OK
        ```

#### **🌞 Tester le DNS depuis `marcel.client1.tp3`**
- **Définissez manuellement l'utilisation de votre serveur DNS :**
    ```
    [yrlan@marcel ~]$ cat /etc/resolv.conf
    # Generated by NetworkManager
    search client1.tp3
    nameserver 10.3.0.253 
    ```

- **Essayez une résolution de nom avec dig**    
    - **dig `dhcp.client1.tp3` :**
    ```
    [yrlan@marcel ~]$ dig dhcp.client1.tp3 @10.3.0.253

    ; <<>> DiG 9.11.26-RedHat-9.11.26-4.el8_4 <<>> dhcp.client1.tp3 @10.3.0.253
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 8582
    ;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 1232
    ; COOKIE: 2c05506ff15b1d3eba098f086154c2d73eb0f919d3d93f0b (good)
    ;; QUESTION SECTION:
    ;dhcp.client1.tp3.              IN      A

    ;; ANSWER SECTION:
    dhcp.client1.tp3.       86400   IN      A       10.3.0.61

    ;; AUTHORITY SECTION:
    client1.tp3.            86400   IN      NS      dns1.server1.tp3.

    ;; ADDITIONAL SECTION:
    dns1.server1.tp3.       86400   IN      A       10.3.0.253

    ;; Query time: 3 msec
    ;; SERVER: 10.3.0.253#53(10.3.0.253)
    ;; WHEN: Wed Sep 29 21:47:33 CEST 2021
    ;; MSG SIZE  rcvd: 132
    ```
    
    - **dig `dns1.server1.tp3` :**
    ```
    [yrlan@marcel ~]$ dig dns1.server1.tp3 @10.3.0.253

    ; <<>> DiG 9.11.26-RedHat-9.11.26-4.el8_4 <<>> dns1.server1.tp3 @10.3.0.253
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 8928
    ;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 1232
    ; COOKIE: 2e6c0c6b9e658ac28b39fc956154c2c5d64e09c0ee1a4102 (good)
    ;; QUESTION SECTION:
    ;dns1.server1.tp3.              IN      A

    ;; ANSWER SECTION:
    dns1.server1.tp3.       86400   IN      A       10.3.0.253

    ;; AUTHORITY SECTION:
    server1.tp3.            86400   IN      NS      dns1.server1.tp3.

    ;; Query time: 1 msec
    ;; SERVER: 10.3.0.253#53(10.3.0.253)
    ;; WHEN: Wed Sep 29 21:47:15 CEST 2021
    ;; MSG SIZE  rcvd: 103
    ```
    
- **Prouvez que c'est bien votre serveur DNS qui répond pour chaque dig**
    - **Pour chaque requête Dig, j'ai utilisé `dig <hostname> @10.3.0.253` et on a bien dans la ligne qui indique le serveur utilisé :**
    ```
    ;; SERVER: 10.3.0.253#53(10.3.0.253)
    ```
    
    - **On peut également utiliser la commande `nslookup` et on voit le serveur DNS utilisé ainsi que l'IP correspondant au nom demandé :** 
    ```
    [yrlan@marcel ~]$ nslookup dhcp.client1.tp3
    Server:         10.3.0.253
    Address:        10.3.0.253#53

    Name:   dhcp.client1.tp3
    Address: 10.3.0.61
    --------------------------------------------
    [yrlan@marcel ~]$ nslookup dns1.server1.tp3
    Server:         10.3.0.253
    Address:        10.3.0.253#53

    Name:   dns1.server1.tp3
    Address: 10.3.0.253
    ```

#### **🌞 Configurez l'utilisation du serveur DNS sur TOUS vos noeuds**
- **Les serveurs, on le fait à la main**
    > **En ajoutant une ligne DNS1=10.3.0.253 dans le fichier de conf de l'interface réseau**
    ```   
    [yrlan@dhcp ~]$ sudo cat /etc/sysconfig/network-scripts/ifcfg-enp0s8 | grep DNS1
    DNS1=10.3.0.253

    [yrlan@dns1 ~]$ sudo cat /etc/sysconfig/network-scripts/ifcfg-enp0s8 | grep DNS1
    DNS1=10.3.0.253
    ```

- **Les clients, c'est fait via DHCP**
    > **On change le `option domain-name-servers` + reboot service DHCP**
    ```
    [yrlan@dhcp ~]$ sudo cat /etc/dhcp/dhcpd.conf | grep "option domain-name-servers"
        option domain-name-servers      10.3.0.253;
    [yrlan@dhcp ~]$ sudo systemctl restart dhcpd
    
    [yrlan@marcel ~]$ sudo dhclient -r; sudo dhclient
    [yrlan@marcel ~]$ sudo cat /etc/resolv.conf
    ; generated by /usr/sbin/dhclient-script
    nameserver 10.3.0.253
    
    ```
## **3. Get deeper**

### **A. DNS forwarder**
#### **🌞 Affiner la configuration du DNS**

- **Faites en sorte que votre DNS soit désormais aussi un forwarder DNS**
    - **C'est à dire que s'il ne connaît pas un nom, il ira poser la question à quelqu'un d'autre**
    ```
    [yrlan@dns1 ~]$ sudo cat /etc/named.conf | grep recursion
        recursion       yes;
    ```

#### **🌞 Test !**

- **Vérifier depuis marcel.client1.tp3 que vous pouvez résoudre des noms publics comme google.com en utilisant votre propre serveur DNS (commande dig)**
    - **Pour que ça fonctionne, il faut que dns1.server1.tp3 soit lui-même capable de résoudre des noms, avec 1.1.1.1 par exemple**
    ```
    [yrlan@marcel ~]$ dig google.com

    ; <<>> DiG 9.11.26-RedHat-9.11.26-4.el8_4 <<>> google.com
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 47566
    ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 4, ADDITIONAL: 9

    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 1232
    ; COOKIE: 35278fae4720709d9ebc00106154efd4e13cf8b037c91443 (good)
    ;; QUESTION SECTION:
    ;google.com.                    IN      A

    ;; ANSWER SECTION:
    google.com.             300     IN      A       142.250.75.238

    ;; AUTHORITY SECTION:
    google.com.             172800  IN      NS      ns3.google.com.
    google.com.             172800  IN      NS      ns4.google.com.
    google.com.             172800  IN      NS      ns2.google.com.
    google.com.             172800  IN      NS      ns1.google.com.

    ;; ADDITIONAL SECTION:
    ns2.google.com.         172800  IN      A       216.239.34.10
    ns1.google.com.         172800  IN      A       216.239.32.10
    ns3.google.com.         172800  IN      A       216.239.36.10
    ns4.google.com.         172800  IN      A       216.239.38.10
    ns2.google.com.         172800  IN      AAAA    2001:4860:4802:34::a
    ns1.google.com.         172800  IN      AAAA    2001:4860:4802:32::a
    ns3.google.com.         172800  IN      AAAA    2001:4860:4802:36::a
    ns4.google.com.         172800  IN      AAAA    2001:4860:4802:38::a

    ;; Query time: 270 msec
    ;; SERVER: 10.3.0.253#53(10.3.0.253)
    ;; WHEN: Thu Sep 30 00:59:31 CEST 2021
    ;; MSG SIZE  rcvd: 331
    ```

### **B. On revient sur la conf du DHCP**
> **🖥️ VM `johnny.client1.tp3`**

#### **🌞 Affiner la configuration du DHCP**
- **Faites en sorte que votre DHCP donne désormais l'adresse de votre serveur DNS aux clients**
    ```
    [yrlan@dhcp ~]$ sudo cat /etc/dhcp/dhcpd.conf | grep "option domain-name-servers"
            option domain-name-servers      10.3.0.253;
    ```
- **Créer un nouveau client `johnny.client1.tp3` qui récupère son IP, et toutes les nouvelles infos, en DHCP**

> **Pour cette partie j'avais fait un `dhclient` dans VirtualBox pour me connecter en SSH mais j'avais encore le hostname `patron`, puis j'ai fais le `dhclient -r; dhclient -v`**

> **C'est pour ça que l'IP obtenue est 10.3.0.13 (.12 était réservée pour le hostname `patron`, j'ai reperé ça dans le fichier `/var/lib/dhcpd/dhcpd.leases`**

```
[yrlan@patron ~]$ sudo dhclient 
PS C:\Users\yrlan> ssh 10.3.0.12
[yrlan@patron ~]$ echo 'johnny.client1.tp3' | sudo tee /etc/hostname; sudo reboot now

[yrlan@johnny ~]$ sudo dhclient -r; sudo dhclient -v
[sudo] password for yrlan:
Internet Systems Consortium DHCP Client 4.3.6
Copyright 2004-2017 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/

Listening on LPF/enp0s8/08:00:27:b2:f4:9a
Sending on   LPF/enp0s8/08:00:27:b2:f4:9a
Sending on   Socket/fallback
DHCPDISCOVER on enp0s8 to 255.255.255.255 port 67 interval 8 (xid=0x404db54)
DHCPDISCOVER on enp0s8 to 255.255.255.255 port 67 interval 10 (xid=0x404db54)
DHCPREQUEST on enp0s8 to 255.255.255.255 port 67 (xid=0x404db54)
DHCPOFFER from 10.3.0.61
DHCPACK from 10.3.0.61 (xid=0x404db54)
bound to 10.3.0.13 -- renewal in 36802 seconds.

[yrlan@johnny ~]$ ip r s
default via 10.3.0.62 dev enp0s8
10.3.0.0/26 dev enp0s8 proto kernel scope link src 10.3.0.13

[yrlan@johnny ~]$ sudo nmcli device show enp0s8
[...]
IP4.ADDRESS[1]:                         10.3.0.13/26
IP4.GATEWAY:                            10.3.0.62
IP4.DNS[1]:                             10.3.0.253
[...]
```

# **Entracte**
**A ce stade vous avez :**

- **Un routeur qui permet aux machines d'acheminer leur trafic entre les réseaux :+1:**
    - **Entre les LANs :+1:**
    - **Vers internet :+1:**
    > **Vérifié avec `ping 8.8.8.8` + `ping` entre les VM**


- **Un DHCP qui filent des infos à vos clients :+1:**
    - **Une IP, une route par défaut, l'adresse d'un DNS :+1:**
    > **Vérifié avec `sudo nmcli device show enp0s8`**

- **Un DNS :+1:**
    - **Qui résout tous les noms localement :+1:**
    > **Vérifié avec `dig` et `nslookup`** 

# **III. Services métier**

## **1. Serveur Web**
> **🖥️ VM `web1.server2.tp3`**

#### **🌞 Setup d'une nouvelle machine, qui sera un serveur Web, une belle appli pour nos clients**

- **Réseau `server2`**
    ```
    [yrlan@nfs1 ~]$ sudo cat /etc/sysconfig/network-scripts/ifcfg-enp0s8
    TYPE=Ethernet
    BOOTPROTO=static
    IPADDR=10.3.1.3
    NETMASK=255.255.255.240
    DEFROUTE=yes
    NAME=enp0s8
    UUID=c077c523-fb65-4a2a-8998-c40b1466e030
    DEVICE=enp0s8
    ONBOOT=yes
    GATEWAY=10.3.1.14
    DNS1=10.3.0.253
    
    [yrlan@nfs1 ~]$ ip r s
    default via 10.3.1.14 dev enp0s8 proto static metric 100
    10.3.1.0/28 dev enp0s8 proto kernel scope link src 10.3.1.3 metric 100

    [yrlan@web1 ~]$ sudo nmcli device show enp0s8
    IP4.ADDRESS[1]:                         10.3.1.2/28
    IP4.GATEWAY:                            10.3.1.14
    IP4.DNS[1]:                             10.3.0.253
    
    
    [yrlan@web1 ~]$ cat /etc/resolv.conf
    # Generated by NetworkManager
    search server2.tp3
    nameserver 10.3.0.253
    
    [yrlan@web1 ~]$ sudo cat /etc/sysconfig/network
    # Created by anaconda
    GATEWAY=10.3.1.14
    ```
    
- **Hello `web1.server2.tp3` !**
    ```
    [yrlan@web1 ~]$ sudo hostname
    web1.server2.tp3
    ```

- **Vous utiliserez le serveur web que vous voudrez, le but c'est d'avoir un serveur web fast, pas d'y passer 1000 ans :)**
    - **Réutilisez votre serveur Web du TP1 Linux**
    ```
    # Création du serveur python
    [yrlan@web1 ~]$ sudo vim /etc/systemd/system/web.service
    [yrlan@web1 ~]$ cat /etc/systemd/system/web.service
    [Unit]
    Description=Very simple web service

    [Service]
    ExecStart=/bin/python3 -m http.server 8888

    [Install]
    WantedBy=multi-user.target
    
    # On relance la conf systemd (pour que le fichier web.service soit découvert)
    [yrlan@web1 ~]$ sudo systemctl daemon-reload

    # Lancement + Lancement au démarrage du service web
    [yrlan@web1 ~]$ sudo systemctl start web; sudo systemctl enable web
    Created symlink /etc/systemd/system/multi-user.target.wants/web.service → /etc/systemd/system/web.service.
    ```
    
    - **Dans tous les cas, n'oubliez pas d'ouvrir le port associé dans le firewall pour que le serveur web soit joignable**
    ```
    # Ouverture du port 8888/tcp défini dans /etc/systemd/system/web.service
    
    [yrlan@web1 ~]$ sudo firewall-cmd --add-port=8888/tcp --permanent; sudo firewall-cmd --reload; sudo firewall-cmd --list-all
    success
    success
    public (active)
      target: default
      icmp-block-inversion: no
      interfaces: enp0s8
      sources:
      services: ssh
      ports: 8888/tcp
      protocols:
      masquerade: no
      forward-ports:
      source-ports:
      icmp-blocks:
      rich rules:
    ```

#### **🌞 Test test test et re-test**
- **Testez que votre serveur `web1.server2.tp3` est accessible depuis `marcel.client1.tp3`**
    - **Utilisez la commande `curl` pour effectuer des requêtes HTTP**
    ```
    [yrlan@dns1 ~]$ sudo nano /var/named/server2.tp3.forward
    [sudo] password for yrlan:
    [yrlan@dns1 ~]$ sudo cat /var/named/server2.tp3.forward | grep web1
    web1    IN A    10.3.1.2
    
    [yrlan@marcel ~]$ curl web1.server2.tp3:8888
    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <title>Directory listing for /</title>
    </head>
    <body>
    <h1>Directory listing for /</h1>
    <hr>
    <ul>
    <li><a href="bin/">bin@</a></li>
    <li><a href="boot/">boot/</a></li>
    <li><a href="dev/">dev/</a></li>
    <li><a href="etc/">etc/</a></li>
    <li><a href="home/">home/</a></li>
    <li><a href="lib/">lib@</a></li>
    <li><a href="lib64/">lib64@</a></li>
    <li><a href="media/">media/</a></li>
    <li><a href="mnt/">mnt/</a></li>
    <li><a href="opt/">opt/</a></li>
    <li><a href="proc/">proc/</a></li>
    <li><a href="root/">root/</a></li>
    <li><a href="run/">run/</a></li>
    <li><a href="sbin/">sbin@</a></li>
    <li><a href="srv/">srv/</a></li>
    <li><a href="sys/">sys/</a></li>
    <li><a href="tmp/">tmp/</a></li>
    <li><a href="usr/">usr/</a></li>
    <li><a href="var/">var/</a></li>
    </ul>
    <hr>
    </body>
    </html>
    ```

## **2. Partage de fichiers**
### **A. L'introduction wola**
### **B. Le setup wola**
> **🖥️ VM `nfs1.server2.tp3`**
#### **🌞 Setup d'une nouvelle machine, qui sera un serveur NFS**

- **Réseau `server2`**
    ```
    [yrlan@nfs1 ~]$ sudo cat /etc/sysconfig/network-scripts/ifcfg-enp0s8
    TYPE=Ethernet
    BOOTPROTO=static
    IPADDR=10.3.1.3
    NETMASK=255.255.255.240
    DEFROUTE=yes
    NAME=enp0s8
    UUID=c077c523-fb65-4a2a-8998-c40b1466e030
    DEVICE=enp0s8
    ONBOOT=yes
    GATEWAY=10.3.1.14
    DNS1=10.3.0.253
    
    [yrlan@nfs1 ~]$ sudo nmcli device show enp0s8
    [...]
    IP4.ADDRESS[1]:                         10.3.1.3/28
    IP4.GATEWAY:                            10.3.1.14
    IP4.DNS[1]:                             10.3.0.253
    [...]
    ```

- **Son nooooom : `nfs1.server2.tp3` !**
    ```
    [yrlan@nfs1 ~]$ hostname
    `nfs1.server2.tp3`
    
    [yrlan@dns1 ~]$ sudo cat /var/named/server2.tp3.forward | grep nfs1
    nfs1    IN A    10.3.1.3
    ```

- **Je crois que vous commencez à connaître la chanson... Google "nfs server rocky linux"**
- **Vous partagerez un dossier créé à cet effet : `/srv/nfs_share/`**
    ```
    [yrlan@nfs1 ~]$ sudo dnf install -y nfs-utils
    [yrlan@nfs1 ~]$ sudo nano /etc/idmapd.conf
    [yrlan@nfs1 ~]$ sudo cat /etc/idmapd.conf | grep Domain
    Domain = server2.tp3

    [yrlan@nfs1 ~]$ sudo mkdir /srv/nfs_share/
    [yrlan@nfs1 ~]$ sudo nano /etc/exports
    [yrlan@nfs1 ~]$ sudo cat /etc/exports
    /srv/nfs_share 10.3.1.0/28(rw,no_root_squash)
    
    # Autorisation NFS dans le pare-feu
    [yrlan@nfs1 ~]$ sudo firewall-cmd --add-service=nfs --permanent; sudo firewall-cmd --reload; sudo firewall-cmd --list-all
    success
    success
    public (active)
      target: default
      icmp-block-inversion: no
      interfaces: enp0s8
      sources:
      services: nfs ssh
      ports:
      protocols:
      masquerade: no
      forward-ports:
      source-ports:
      icmp-blocks:
      rich rules:
      
      [yrlan@nfs1 ~]$ systemctl enable --now rpcbind nfs-server
    ```
#### **🌞 Configuration du client NFS**

- **Effectuez de la configuration sur `web1.server2.tp3` pour qu'il accède au partage NFS**
    ```
    [yrlan@web1 ~]$ sudo dnf -y install nfs-utils
    [yrlan@web1 ~]$ sudo nano /etc/idmapd.conf
    [yrlan@web1 ~]$ sudo cat /etc/idmapd.conf | grep Domain
    Domain = server2.tp3    
    ```

- **Le partage NFS devra être monté dans `/srv/nfs/`**
    ```
    [yrlan@web1 ~]$ sudo mount -t nfs nfs1.server2.tp3:/srv/nfs_share /srv/nfs
    [yrlan@web1 ~]$ df -hT
    Filesystem                      Type      Size  Used Avail Use% Mounted on
    devtmpfs                        devtmpfs  387M     0  387M   0% /dev
    tmpfs                           tmpfs     405M     0  405M   0% /dev/shm
    tmpfs                           tmpfs     405M  5.6M  400M   2% /run
    tmpfs                           tmpfs     405M     0  405M   0% /sys/fs/cgroup
    /dev/mapper/rl-root             xfs       6.2G  2.2G  4.1G  35% /
    /dev/sda1                       xfs      1014M  301M  714M  30% /boot
    tmpfs                           tmpfs      81M     0   81M   0% /run/user/1000
    nfs1.server2.tp3:/srv/nfs_share nfs4      6.2G  2.2G  4.1G  35% /srv/nfs
    ```

#### **🌞 TEEEEST**

- **Tester que vous pouvez lire et écrire dans le dossier `/srv/nfs` depuis `web1.server2.tp3`**
    ```
    [yrlan@web1 ~]$ cd /srv/nfs
    [yrlan@web1 nfs]$ sudo mkdir Dossier1
    [yrlan@web1 nfs]$ sudo touch MonserveurNFS.text

    [yrlan@web1 nfs]$ sudo cat MonserveurNFS.text
    J'écris du texte depuis web1.server2.tp3, qui est dans le réseau 10.3.1.0/28 que j'ai précisé dans le fichier idmapd.conf sur ma VM nfs1.server2.tp3

    [yrlan@web1 nfs]$ ls -al
    total 4
    drwxr-xr-x. 3 root root  48 Sep 30 06:17 .
    drwxr-xr-x. 3 root root  17 Sep 30 05:55 ..
    drwxr-xr-x. 2 root root   6 Sep 30 06:17 Dossier1
    -rw-r--r--. 1 root root 152 Sep 30 06:20 MonserveurNFS.text
    ```


- **Vous devriez voir les modifications du côté de `nfs1.server2.tp3` dans le dossier `/srv/nfs_share/`**
    ```
    [yrlan@nfs1 ~]$ cd /srv/nfs_share/
    [yrlan@nfs1 nfs_share]$ ls -al
    total 4
    drwxr-xr-x. 3 root root  48 Sep 30 06:17 .
    drwxr-xr-x. 3 root root  23 Sep 30 05:20 ..
    drwxr-xr-x. 2 root root   6 Sep 30 06:17 Dossier1
    -rw-r--r--. 1 root root 152 Sep 30 06:20 MonserveurNFS.text

    [yrlan@nfs1 nfs_share]$ cat MonserveurNFS.text
    J'écris du texte depuis web1.server2.tp3, qui est dans le réseau 10.3.1.0/28 que j'ai précisé dans le fichier idmapd.conf sur ma VM nfs1.server2.tp3
    ```

# **IV. Un peu de théorie : TCP et UDP**

#### **🌞 Déterminer, pour chacun de ces protocoles, s'ils sont encapsulés dans du TCP ou de l'UDP :**
- **SSH** => Port `22/TCP`
- **HTTP** => Port `80/TCP`
- **DNS** => Par défaut, Port `53/UDP` mais peut utiliser `53/TCP`
- **NFS** => Par défaut, Port `2049/TCP` mais peut utiliser `2049/UDP` 

- **📁 Captures réseau**
	- **[`tp3_ssh.pcap`](./Capture/tp3_ssh.pcap)**
	- **[`tp3_http.pcap`](./Capture/tp3_http.pcap)**
	- **[`tp3_dns.pcap`](./Capture/tp3_dns.pcap)**
	- **[`tp3_nfs.pcap`](./Capture/tp3_nfs.pcap)**

#### **🌞 Capturez et mettez en évidence un 3-way handshake**

- **📁 Capture réseau**
    - **[`tp3_3way.pcap`](./Capture/tp3_3way.pcap)**

# **V. El final**

#### **🌞 Bah j'veux un schéma.**

- **Dans le rendu, mettez moi ici à la fin :**
    - **Le schéma**

    > Je l'ai fait par rapport au réseau que j'aurais du utiliser (voir en dessous)

    ![](./img/Schéma_réseaux.png)

    - **Le 🗃️ tableau des réseaux 🗃️**

    > **Tableau des réseaux que j'ai utilisé pour le TP**

    | Nom du réseau | Adresse du réseau | Masque            | Nombre de clients possibles | Adresse passerelle | Adresse broadcast |
    |---------------|-------------------|-------------------|-----------------------------|--------------------|-----------------|
    | `client1`     | `10.3.0.0`        | `255.255.255.192` | 61                          | `10.3.0.62`        | `10.3.0.63`       |
    | `server1`     | `10.3.0.128`      | `255.255.255.128` | 125                         | `10.3.0.254`       | `10.3.0.255`      |
    | `server2`     | `10.3.1.0`        | `255.255.255.240` | 13                          | `10.3.1.14`        | `10.3.1.15`       |

    > **Tableau des réseaux que j'aurais du utiliser**

    | Nom du réseau | Adresse du réseau | Masque            | Nombre de clients possibles | Adresse passerelle | Adresse broadcast |
    |---------------|-------------------|-------------------|-----------------------------|--------------------|-------------------|
    | `server1`     | `10.3.0.0`        | `255.255.255.128` | 125                         | `10.3.0.126`       | `10.3.0.127`      |
    | `client1`     | `10.3.0.128`      | `255.255.255.192` | 61                          | `10.3.0.190`       | `10.3.0.191`      |
    | `server2`     | `10.3.0.192`      | `255.255.255.240` | 13                          | `10.3.0.206`       | `10.3.0.207`      |

	- **Le 🗃️ tableau d'adressage 🗃️**
	    - **On appelle ça aussi un "plan d'adressage IP" :)**

    > **Tableau d'adressage que j'ai utilisé pour le TP**
            
    | Nom machine           | Adresse IP `client1` | Adresse IP `server1` | Adresse IP `server2` | Adresse de passerelle |
    | --------------------- | -------------------- | -------------------- | -------------------- | --------------------- |
    | `router.tp3`          | `10.3.0.62/26`       | `10.3.0.254/25`      | `10.3.1.14/28`       | Carte NAT             |
    | `dhcp.client1.tp3`    | `10.3.0.61/26`       | ...                  | ...                  | `10.3.0.62/26`        |
    | `marcel.client1.tp3`  | `10.3.0.11/26`       | ...                  | ...                  | `10.3.0.62/26`        |
    | `johnny.client1.tp3`  | `10.3.0.13/26`       | ...                  | ...                  | `10.3.0.62/26`        |
    | `dns1.server1.tp3`    | ...                  | `10.3.0.253/25`      | ...                  | `10.3.0.254/25`       |
    | `web1.server2.tp3`    | ...                  | ...                  | `10.3.1.2/28`        | `10.3.1.14/28 `       |
    | `nfs1.server2.tp3`    | ...                  | ...                  | `10.3.1.3/28`        | `10.3.1.14/28 `       |

    > **Tableau d'adressage que j'aurais pu utiliser sur le 2nd réseau (qui correspond avec le schéma)**

    | Nom machine           | Adresse IP `server1` | Adresse IP `client1` | Adresse IP `server2` | Adresse de passerelle |
    | --------------------- | -------------------- | -------------------- | -------------------- | --------------------- |
    | `router.tp3`          | `10.3.0.126/25`      | `10.3.0.190/26`      | `10.3.0.206/28`      | Carte NAT             |
    | `dhcp.client1.tp3`    | ...                  | `10.3.0.189/26`      | ...                  | `10.3.0.190/26`       |
    | `marcel.client1.tp3`  | ...                  | `10.3.0.131/26`      | ...                  | `10.3.0.190/26`       |
    | `johnny.client1.tp3`  | ...                  | `10.3.0.132/26`      | ...                  | `10.3.0.190/26`       |
    | `dns1.server1.tp3`    | `10.3.0.125/25`      | ...                  | ...                  | `10.3.0.254/25`       |
    | `web1.server2.tp3`    | ...                  | ...                  | `10.3.0.204/28`      | `10.3.0.206/28 `      |
    | `nfs1.server2.tp3`    | ...                  | ...                  | `10.3.0.205/28`      | `10.3.0.206/28 `      |


#### **🌞 Et j'veux des fichiers aussi, tous les fichiers de conf du DNS**

- **📁 Fichiers de zone**
	- [client1.tp3.forward](./conf/client1.tp3.forward)
	- [server1.tp3.forward](./conf/server1.tp3.forward)
	- [server2.tp3.forward](./conf/server2.tp3.forward)

- **📁 Fichier de conf principal DNS named.conf**
    - [named.conf](./conf/named.conf)

- **Faites ça à peu près propre dans le rendu, que j'ai plus qu'à cliquer pour arriver sur le fichier ce serait top** :+1: 
