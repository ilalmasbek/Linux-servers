#--SRV1--
apt install -y bind9 bind9-utils

vim /etcbind/named.conf.options
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

vim /etcbind/named.conf.local
    zone "lab.local" {
            type master;
            file "/etc/bind/db.lab.local";
    };
    
    zone "16.172.in-addr.arpa" {
            type master;
            file "/etc/bind/db.reverse";
    };

cd /etc/bind
cp db.local db.lab.local
cp db.127 db.reverse
