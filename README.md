# Installing Bind on Ubuntu 20.04

1. Make sure there are no DNS servers running - e.g. dnsmasq
```shell
sudo systemctl stop dnsmasq.service
sudo systemctl disable dnsmasq.service
```

2. Install the latest packages and update configurations are working
```shell
sudo apt install bind9
named -V
sudo systemctl start named
sudo systemctl enable named
sudo systemctl status named
sudo netstat -lnptu
sudo rndc status
```

3. Modify the configuration files - named.conf.options - sample below
```
options {
	directory "/var/cache/bind";
	version none;
  dnssec-validation no;
	forwarders {10.142.7.21; 10.142.7.22; 8.8.4.4;};
	listen-on-v6 { none; }; 
  listen-on { 127.0.0.1; 192.168.100.1; 192.168.102.1; 192.168.104.1;};
  recursion yes;
	allow-recursion { 127.0.0.1; 192.168.100.0/23; 192.168.102.0/23; 192.168.104.0/23;};
};
```

4. Create reference to forward and reverse lookup zone files - named.conf.local.
```
zone "env1.lab.test" {
    type master;
    file "/etc/bind/zones/db.env1.lab.test";
};

zone "100.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/db.100.168.192";
};
```

5. Create the forward lookup zone file in the correct folder - e.g. /etc/bind/zones/db.env1.lab.test
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     ubuntu-nv-241.env1.lab.test. admin.env1.lab.test. (
                              9         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
; name servers - NS records
     IN      NS      ubuntu-nv-241.env1.lab.test.

; name servers - A records
ubuntu-nv-241.env1.lab.test.            IN      A       192.168.100.1


harbor.env1.lab.test.			      IN 	CNAME	      ubuntu-nv-241.env1.lab.test.
vcsa1.env1.lab.test.            IN      A       192.168.100.50
esxi-11.env1.lab.test.          IN      A       192.168.100.51
esxi-12.env1.lab.test.          IN      A       192.168.100.52
...
```
