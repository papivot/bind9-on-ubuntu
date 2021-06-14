# Installing Bind on Ubuntu 20.04

1. Make sure there are no DNS servers running - e.g. dnsmasq
```console
sudo systemctl stop dnsmasq.service
sudo systemctl disable dnsmasq.service
```

2. Install the latest packages and update configurations are working
```console
sudo apt install bind9 bind9-doc
named -V
sudo systemctl start named
sudo systemctl enable named
sudo systemctl status named
sudo netstat -lnptu
sudo rndc status
```

3. Modify the configuration files - `/etc/bind/named.conf.options` - sample below. My external DNS is x.x.x.x and y.y.y.y. I only want BIND to listen in on my provate networks - 192.168.100.0/23, 192.168.102.0/23, 192.168.104.0/23
```
options {
	directory "/var/cache/bind";
	version none;
  dnssec-validation no;
	forwarders {x.x.x.x; y.y.y.y; 8.8.4.4;};
	listen-on-v6 { none; }; 
  listen-on { 127.0.0.1; 192.168.100.1; 192.168.102.1; 192.168.104.1;};
  recursion yes;
	allow-recursion { 127.0.0.1; 192.168.100.0/23; 192.168.102.0/23; 192.168.104.0/23;};
};
```

4. Create reference to forward and reverse lookup zone files - `/etc/bind/named.conf.local`. My domain is env1.lab.test and the subnet for that is 192.168.100.0/23
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

5. Create the forward lookup zone file in the correct folder - e.g. `/etc/bind/zones/db.env1.lab.test`. The name of my Bind server is  ubuntu-nv-241.env1.lab.test.
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

6. Create the reverse lookup zone file in the correct folder - For e.g `/etc/bind/zones/db.100.168.192`
```
;
; BIND reverse data file for local loopback interface
;
$TTL	604800
@       IN      SOA     ubuntu-nv-241.env1.lab.test. admin.env1.lab.test. (
			      7		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			 604800 )	; Negative Cache TTL
;
; name servers - NS records
     	NS      ubuntu-nv-241.env1.lab.test.

; PTR Records
1	PTR	ubuntu-nv-241.env1.lab.test.	;192.168.100.1

50  	PTR     vcsa1.env1.lab.test.  	;192.168.100.50
51  	PTR     esxi-11.env1.lab.test.  ;192.168.100.51
52  	PTR     esxi-12.env1.lab.test.  ;192.168.100.52
...
```

7. Restart the Bind service
```console
sudo systemctl restart named
```

Troubleshooting commands - 
```console
sudo named-checkconf
sudo journalctl -eu named
sudo named-checkzone env1.lab.test db.env1.lab.test
sudo named-checkzone 100.168.192.in-addr.arpa db.100.168.192
```

Check the resolution
```console
dig A vcsa1.env1.lab.test @192.168.100.1
dig A vcsa1.env1.lab.test @192.168.102.1
dig -x 192.168.100.50 @192.168.100.1
dig -x 192.168.100.50 @192.168.102.1
...
```
