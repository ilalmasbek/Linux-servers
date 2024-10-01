# Before starting configuration, please read the readme file.
## SRV1 | Step 1: Configure the first network settings.
```shell
nano /etc/network/interfaces
```
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
apt install -y bind9 bind9-utils dnsutils
```
## Step 3: Configure forwarding of DNS requests to the DNS servers of providers.
```shell
vim /etc/bind/named.conf.options
```
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
```shell
vim /etc/bind/named.conf.local
```
```shell
zone "lab.local" {
    type master;
    file "/var/lib/bind/db.lab.local";
    allow-transfer { 172.16.0.7; };
    also-notify { 172.16.0.7; };
};
    
zone "16.172.in-addr.arpa" {
    type master;
    file "/var/lib/bind/db.reverse";
    allow-transfer { 172.16.0.7; };
    also-notify { 172.16.0.7; };
};
```
## Step 6: Copy files in forward and reverse zones into new files, as well in settings in zones.
```shell
cd /etc/bind
```
```shell
cp db.local /var/lib/bind/db.lab.local
```
```shell
cp db.127 /var/lib/bind/db.reverse
```
## Step 7: Forward zone settings.
```shell
nano /var/lib/bind/db.lab.local
```
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
named-checkzone lab.local /var/lib/bind/db.lab.local
```
## Step 9: Reverse zone settings.
```shell
nano /var/lib/bind/db.reverse
```
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
named-checkzone 16.172.in-addr.arpa /var/lib/bind/db.reverse
```
## Step 11: Restart and check the status of the BIND service.
```shell
systemctl restart bind9
```
```shell
systemctl status bind9
```
## Step 12: Reconfigure network settings.
```shell
nano /etc/network/interfaces
```
```shell
auto eth0
iface eth0 inet static
        address 172.16.0.6/24
        gateway 172.16.0.1
        dns-nameservers 172.16.0.6 172.16.0.7
```
```shell
nano /etc/resolv.conf
```
```shell
nameserver 172.16.0.6
nameserver 172.16.0.7
domain lab.local
```
```shell
systemctl restart networking
```
## SRV2 | Step 1: Configure the first network settings.
```shell
nano /etc/network/interfaces
```
```shell
auto eth0
iface eth0 inet static
        address 172.16.0.7/24
        gateway 172.16.0.1
        dns-nameservers 8.8.8.8
```
```shell
systemctl restart networking
```
## Step 2: Install the DNS server package and DNS tools.
```shell
apt install -y bind9 bind9-utils dnsutils
```
## Step 3: Create forward and reverse zones.
```shell
vim /etc/bind/named.conf.local
```
```shell
zone "lab.local" {
    type secondary;
    file "/var/lib/bind/db.lab.local";
    masters { 172.0.0.6; };
};
    
zone "16.172.in-addr.arpa" {
    type secondary;
    file "/var/lib//bind/db.reverse";
    masters { 172.0.0.6; };
};
```
## Step 4: Restart and check the status of the BIND service.
```shell
systemctl restart bind9
```
```shell
systemctl status bind9
```
## Step 5: Reconfigure network settings.
nano /etc/network/interfaces
```shell
auto eth0
iface eth0 inet static
        address 172.16.0.7/24
        gateway 172.16.0.1
        dns-nameservers 172.16.0.6 172.16.0.7
```
```shell
nano /etc/resolv.conf
```
```shell
nameserver 172.16.0.6
nameserver 172.16.0.7
domain lab.local
```
```shell
systemctl restart networking
```
## Additionally
```shell
nslookup srv1
```
```shell
host srv1
```
```shell
journalctl -u bind9
```
```shell
tail -f /var/log/syslog
```
```shell
dig @172.16.0.7 lab.local
```










