# Before starting configuration, please read the readme file. 
## SRV1 | Step 1 
=====================================
cd /etc/bind/
ddns-confgen -k keygen1
nano dns.key
key "keygen1" {
        algorithm hmac-sha256;
        secret "IZeTrtydTlsL7fj4BVyR2x4pSsfoBaVQgN5Qj/T2uIY=";
};
===============================================
nano /etc/bind/named.conf.local
include "/etc/bind/dns.key";

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
=======================================
cd /etc/dhcp/
nano dhcp.key
key "keygen1" {
        algorithm hmac-sha256;
        secret "IZeTrtydTlsL7fj4BVyR2x4pSsfoBaVQgN5Qj/T2uIY=";
};
=============================================
nano dhcpd.conf

ddns-updates on;
ddns-update-style standard;
ddns-domainname "lab.local";
ddns-rev-domainname "16.172.in-addr.arpa";
include "/etc/dhcp/dhcp.key";
zone example.com. {
    primary 172.16.0.6;
    key keygen1;
}
zone 16.172.in-addr.arpa. {
    primary 172.16.0.6;
    key keygen1;
}
======================================
systemctl restart bind9
systemctl status bind9

systemctl restart isc-dhcp-server
systemctl status isc-dhcp-server
======================================
dhcp-lease-list 
tail -f /var/log/syslog
more /var/lib/bind/db.lab.local



