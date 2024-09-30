# Before starting configuration, please read the readme file. 
## SRV1 | Step 1 
=====================================
cd /var/lib/bind/
```shell
ddns-confgen -k keygen1
```
```shell
nano dns.key
```
```shell
key "keygen1" {
        algorithm hmac-sha256;
        secret "IZeTrtydTlsL7fj4BVyR2x4pSsfoBaVQgN5Qj/T2uIY=";
};
```
===============================================
## Step 2 
nano /etc/bind/named.conf.local
```shell
include "/var/lib/bind/dns.key";

zone "lab.local" {
    type master;
    ...    
    update-policy {
        grant keygen1 wildcard *.lab.local A DHCID;
    };
};

zone "16.172.in-addr.arpa" {
    type master;
    ...
    update-policy {
            grant keygen1 wildcard *.16.172.in-addr.arpa PTR DHCID;
    };
};
```
=======================================
## Step 3 
```shell
cd /etc/dhcp/
nano dhcp.key
```
```shell
key "keygen1" {
        algorithm hmac-sha256;
        secret "IZeTrtydTlsL7fj4BVyR2x4pSsfoBaVQgN5Qj/T2uIY=";
};
```
=============================================
## Step 4 
nano dhcpd.conf
```shell
ddns-updates on;
ddns-update-style standard;
ddns-domainname "lab.local";
ddns-rev-domainname "16.172.in-addr.arpa";
include "/etc/dhcp/dhcp.key";
zone lab.local. {
    primary 172.16.0.6;
    key keygen1;
}
zone 16.172.in-addr.arpa. {
    primary 172.16.0.6;
    key keygen1;
}
```
======================================
## Step 6
```shell
systemctl restart bind9
systemctl status bind9
```
```shell
systemctl restart isc-dhcp-server
systemctl status isc-dhcp-server
```
======================================
## SRV2 | Step 1








```shell
dhcp-lease-list
```
```shell
tail -f /var/log/syslog
```
```shell
more /var/lib/bind/db.lab.local
```

