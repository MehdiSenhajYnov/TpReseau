# TP4 : TCP, UDP et services réseau

# I. First steps

🌞 **Déterminez, pour ces 5 applications, si c'est du TCP ou de l'UDP**

- UDP :
  - Discord
  - Teamviewer
- TCP :
  - MicrosoftStore
  - Youtube
  - Twitch

🌞 **Demandez l'avis à votre OS**

🦈 **[`Discord`](./Discord.pcapng)**
🦈 **[`Teamviewer`](./Teamviewer.pcapng)**
🦈 **[`Twitch`](./Twitch.pcapng)**
🦈 **[`Youtube`](./Youtube.pcapng)**
🦈 **[`MicrosoftStore`](./MicrosoftStore.pcapng)**

# II. Mise en place

## 1. SSH

🌞 **Examinez le trafic dans Wireshark**

```
Le trafic est avec un protocole TCP
```

🌞 **Demandez aux OS**

- repérez, avec un commande adaptée, la connexion SSH depuis votre machine
- ET repérez la connexion SSH depuis votre VM

```
C:\Windows\system32> netstat -fbn -p TCP 5
Connexions actives
  Proto  Adresse locale         Adresse distante       État
  TCP    192.168.1.81:52310         10.3.1.2:22            ESTABLISHED
 [ssh.exe]
```

```
[mehdi@reseau ~]$ sudo ss -ltpn
State      Recv-Q     Send-Q         Local Address:Port         Peer Address:Port    Process
LISTEN     0          128                  0.0.0.0:22                0.0.0.0:*        users:(("sshd",pid=699,fd=3))
LISTEN     0          128                     [::]:22                   [::]:*        users:(("sshd",pid=699,fd=4))
```

## 2. NFS

🌞 **Mettez en place un petit serveur NFS sur l'une des deux VMs**

```
[mehdi@localhost ~]$ sudo dnf -y install nfs-utils
[...]
Complete!
[mehdi@localhost ~]$ sudo nano /etc/idmapd.conf
[mehdi@localhost ~]$ sudo nano /etc/exports
[mehdi@localhost ~]$ cat /etc/exports
/srv/nfsshare 10.3.1.0/24(rw,no_root_squash)
[mehdi@localhost ~]$ sudo systemctl enable --now rpcbind nfs-server
Created symlink /etc/systemd/system/multi-user.target.wants/nfs-server.service → /usr/lib/systemd/system/nfs-server.service.
[mehdi@localhost ~]$ sudo firewall-cmd --add-service=nfs
success
[mehdi@localhost ~]$ sudo firewall-cmd --runtime-to-permanent
success
[mehdi@localhost ~]$ sudo mkdir /srv/nfsshare
[mehdi@localhost ~]$ sudo nano /srv/nfsshare/helloworld.txt
[mehdi@localhost ~]$ cat /srv/nfsshare/helloworld.txt
helloworld!
```

```
[mehdi@localhost ~]$ sudo dnf -y install nfs-utils
[...]
Complete!
[mehdi@localhost ~]$ sudo nano /etc/idmapd.conf
[mehdi@localhost ~]$ sudo mount -t nfs 10.3.1.3:/srv/nfsshare /mnt
[mehdi@localhost ~]$ df -hT /mnt
Filesystem             Type  Size  Used Avail Use% Mounted on
10.3.1.3:/srv/nfsshare nfs4  6.2G  1.2G  5.1G  18% /mnt
[mehdi@localhost ~]$ cat /mnt/helloworld.txt
helloworld!
```

🌞 **Wireshark it !**


Le serveur écoute sur le port 2046.

🌞 **Demandez aux OS**

```
[mehdi@localhost ~]$ sudo ss -ltpn | grep 2046
LISTEN 0      64           0.0.0.0:2046       0.0.0.0:*
LISTEN 0      64              [::]:2046          [::]:*
```

```
[mehdi@localhost ~]$ sudo ss -tp
State                Recv-Q                Send-Q                                Local Address:Port                                 Peer Address:Port                 Process
ESTAB                0                     0                                          10.3.1.2:937                                      10.3.1.3:nfs
```

## 3. DNS

🌞 Utilisez une commande pour effectuer une requête DNS depuis une des VMs

- capturez le trafic avec un `tcpdump`
- déterminez le port et l'IP du serveur DNS auquel vous vous connectez
  - IP : 8.8.8.8
  - Port : 53

```
[mehdi@localhost ~]$ dig ynov.com
; <<>> DiG 9.16.23-RH <<>> ynov.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 37165
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1
;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;ynov.com.                      IN      A
;; ANSWER SECTION:
ynov.com.               300     IN      A       172.67.74.226
ynov.com.               300     IN      A       104.26.11.233
ynov.com.               300     IN      A       104.26.10.233
;; Query time: 30 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Sat Oct 15 17:32:33 CEST 2022
;; MSG SIZE  rcvd: 85
```

