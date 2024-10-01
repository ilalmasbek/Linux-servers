# Before starting configuration, please read the readme file. 
## SRV1 | Step 1 Generate an HMAC key for the DDNS server and copy it. Then create a new file for dns config and paste our copied key there.
```shell
cd /var/lib/bind/
```
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
## Step 2 In the zones file we specify path to the key and allow updates to the forward and reverse zone records.
```shell
nano /etc/bind/named.conf.local
```
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
## Step 3 Create a new file for dhcp config and paste our copied key there.
```shell
cd /etc/dhcp/
```
```shell
nano dhcp.key
```
```shell
key "keygen1" {
        algorithm hmac-sha256;
        secret "IZeTrtydTlsL7fj4BVyR2x4pSsfoBaVQgN5Qj/T2uIY=";
};
```
=============================================
## Step 4 We indicate important parameters, such as the path to the dhcp key, dns zones etc.
```shell
nano dhcpd.conf
```
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
## Step 6 Restart DNS and DHCP services
```shell
systemctl restart bind9
```
```shell
systemctl restart isc-dhcp-server
```
## Step 7 Check the status of DNS and DHCP services to ensure it is running without issues.
```shell
systemctl status bind9
```
```shell
systemctl status isc-dhcp-server
```
======================================
## SRV2 | Step 1 Create a new file for dhcp config and paste our copied key there.
```shell
cd /etc/dhcp/
```
```shell
nano dhcp.key
```
```shell
key "keygen1" {
        algorithm hmac-sha256;
        secret "IZeTrtydTlsL7fj4BVyR2x4pSsfoBaVQgN5Qj/T2uIY=";
};
```
## Step 2 Write the same commands as for the primary DHCP server
```shell
nano dhcpd.conf
```
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
## Step 3 Restart the DHCP Service 
```shell
systemctl restart isc-dhcp-server
```
## Step 4 Check the status of the DHCP service to ensure it is running without issues.
```shell
systemctl status isc-dhcp-server
```
## Additionally
```shell
dhcp-lease-list
```
```shell
tail -f /var/log/syslog
```
```shell
more /var/lib/bind/db.lab.local
```

