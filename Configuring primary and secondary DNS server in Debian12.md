# SRV1
## Step 1: Configure the first network settings.
nano /etc/network/interfaces
```shell
auto eth0
iface eth0 inet static
        address 172.16.0.6/24
        gateway 172.16.0.1
        dns-nameservers 8.8.8.8
```
```shell
systemctl restart networking
```

## Step 2: Install the DNS server package and DNS tools.
```shell
apt install -y bind9 bind9-utils
```
## Step 3: Configure forwarding of DNS requests to the DNS servers of providers.
vim /etc/bind/named.conf.options
```shell
acl mysubnets {
    localhost;
    localnets;
    172.16.0.0/24;
    172.16.1.0/24;
};
    
options {
    directory "/var/cache/bind";
    
    recursion yes;
    allow-query { mysubnets; };
    allow-query-cache { mysubnets; };
    allow-recursion { mysubnets; };
    
    forwarders {
        8.8.8.8;
        8.8.4.4;
    };
    
    dnssec-validation no;
    
    listen-on-v6 { any; };
    listen-on port 53 { 127.0.0.1; 172.16.0.6; };
    };
```
## Step 4: Check the syntax of all BIND configuration files.
```shell
named-checkconf
```
## Step 5: Create forward and reverse zones.
vim /etc/bind/named.conf.local
```shell
zone "lab.local" {
    type master;
    file "/etc/bind/db.lab.local";
};
    
zone "16.172.in-addr.arpa" {
    type master;
    file "/etc/bind/db.reverse";
};
```
## Step 6: Copy files in forward and reverse zones into new files, as well in settings in zones.
```shell
cd /etc/bind
cp db.local db.lab.local
cp db.127 db.reverse
```
## Step 7: Forward zone settings.
nano /etc/bind/db.lab.local
```shell
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     srv1.lab.local. admin.lab.local. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      srv1.lab.local.
srv1    IN      A       172.16.0.6
srv2    IN      A       172.16.0.7
```
## Step 8: Check syntax of the forward zone.
```shell
named-checkzone lab.local /etc/bind/db.lab.local
```
## Step 9: Reverse zone settings.
nano /etc/bind/db.reverse
```shell
;
; BIND reverse data file for local loopback interface
;
$TTL    604800
@       IN      SOA     srv1.lab.local. admin.lab.local. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      srv1.lab.local.
6.0     IN      PTR     srv1.lab.local.
7.0     IN      PTR     srv2.lab.local.
```
## Step 10: Check syntax of the reverse zone.
```shell
named-checkzone 16.172.in-addr.arpa /etc/bind/db.reverse
```
## Step 11: Restart and check the status of the BIND service
```shell
systemctl restart bind9
systemctl status bind9
```

