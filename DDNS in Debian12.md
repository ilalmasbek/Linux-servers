# Before starting configuration, please read the readme file. 
## SRV1 | Step 1 

cd /etc/bind/
dnssec-keygen -a RSASHA256 -b 2048 -n ZONE dhcp-update
ls -l 
-rw-r--r-- 1 root bind  606 Sep 30 11:09  Kdhcp-update.+008+36799.key
-rw------- 1 root bind 1776 Sep 30 11:09  Kdhcp-update.+008+36799.private

