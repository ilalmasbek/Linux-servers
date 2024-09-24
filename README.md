# Linux-servers
#--SRV1--
##Step 1: Install the DHCP Server Package
apt update && upgrade -y
apt install -y isc-dhcp-server

##Step 2: Specify the Network Interface for the DHCP Server
vim /etc/default/isc-dhcp-server
    INTERFACESv4="eth0"

##Step 3: Configure the main config
vim /etc/dhcp/dhcpd.conf
    authoritative;
    shared-network "clients-network" {
    
            failover peer "dhcp-failover" {
                    primary;
                    address 172.16.0.6;
                    port 519;
                    peer address 172.16.0.7;
                    peer port 519;
                    max-response-delay 60;
                    max-unacked-updates 10;
                    mclt 3600;
                    split 128;
                    load balance max seconds 3;
            }
    
            subnet 192.168.1.0 netmask 255.255.255.0 {
                    option routers 192.168.1.1;
                    option domain-name-servers 8.8.8.8, 8.8.4.4;
                    option domain-name "lab.local";
                    option subnet-mask 255.255.255.0;
                    default-lease-time 600;
                    max-lease-time 7200;
            }
    
                    pool {
                            failover peer "dhcp-failover";
                            range 192.168.1.100 192.168.1.200;
                    }
    
            subnet 172.16.0.0 netmask 255.255.255.0 {
            }
    }

##Step 4: Restart and Enable the DHCP Service
systemctl restart isc-dhcp-server
systemctl enable isc-dhcp-server

#--SRV2--
#Step 1: Install the DHCP Server Package
apt update && upgrade -y
apt install -y isc-dhcp-server

##Step 2: Specify the Network Interface for the DHCP Server
vim /etc/default/isc-dhcp-server
    INTERFACESv4="eth0"

##Step 3: Configure the main config
vim /etc/dhcp/dhcpd.conf
    authoritative;
    shared-network "clients-network" {
    
            failover peer "dhcp-failover" {
                    secondary;
                    address 172.16.0.7;
                    port 519;
                    peer address 172.16.0.6;
                    peer port 519;
                    max-response-delay 60;
                    max-unacked-updates 10;
                    mclt 3600;
            }
    
            subnet 192.168.1.0 netmask 255.255.255.0 {
                    option routers 192.168.1.1;
                    option domain-name-servers 8.8.8.8, 8.8.4.4;
                    option domain-name "lab.local";
                    option subnet-mask 255.255.255.0;
                    default-lease-time 600;
                    max-lease-time 7200;
                    pool {
                            failover peer "dhcp-failover";
                            range 192.168.1.100 192.168.1.200;
                    }
            }
            subnet 172.16.0.0 netmask 255.255.255.0 {
            }
    }

##Step 4: Restart and Enable the DHCP Service
systemctl restart isc-dhcp-server
systemctl enable isc-dhcp-server

#Check for any additional configuration errors using:
sudo dhcpd -t -cf /etc/dhcp/dhcpd.conf

#Check the status of the DHCP service to ensure it is running without issues.
systemctl status isc-dhcp-server

#To view the list of clients that have received IP addresses from the DHCP server, you can check the leases file:
cat /var/lib/dhcp/dhcpd.leases

#If you encounter any issues, you can check the logs with:
journalctl -xe | grep dhcp
