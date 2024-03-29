



# Authoritative-Only BIND DNS Server on Ubuntu 18.04



The master server holds the zone files for answering queries, slave comes in to play as a backup server in case of emergencies or system failures. 

``
sudo apt-get update
``

``
sudo apt-get upgrade
``

Bind uses port 53 for zone file transfers make sure you have port 53 allowed on both machines.

Bind is used to make our server a DNS server.

`
sudo ufw allow 53
`

Next proceed to install BIND and its utilities to both DNS machines.

`
sudo apt-get install bind9 bind9utils bind9-doc
`

Allow ipv4 on both machines.

`
sudo nano /etc/default/bind9
`

inside the file add '-4'

``
OPTIONS="-4 -u bind"
``

## Options file.

In order to make our DNS server authoritative only which means the server won't answer to queries that it isn't responsible of, so no recursion.


Allow transfer disables zone transfer requests.

`
sudo nano /etc/bind/named.conf.options
`

Inside the file add ones with '+' mark in front of them.


```
options {
        directory "/var/cache/bind";
       +recursion no;
       +allow-transfer { none; };
        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        // forwarders {
        //      0.0.0.0;
        // };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        dnssec-validation auto;

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
};
```

# Master Configuration

## Local files.

Local files are the files to tell our Master and Slave servers about who to send the zone files to.

`
sudo nano /etc/bind/named.conf.local
`

```
//
// Do any local configuration here
//
zone "iskenhub.info" {
    type master;
    file "/etc/bind/zones/db.iskenhub.info";
    allow-transfer { 192.168.56.102; };
};
zone "56.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/db.34.90";
};


// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

```

Here we defined both the forward and reverse zone files to be transferred to the slave machine.

Now we have to create those zone files and configure them.

`
sudo mkdir /etc/bind/zones
`

`
sudo cp /etc/bind/db.db.127 /etc/bind/zones/db.34.90
`

`
sudo cp /etc/bind/db.local /etc/bind/zones/db.iskenhub.info
`

db.127 and db.local are the pre-made templates for us to work on them.

`
sudo nano /etc/bind/zones/db.iskenhub.info
`

Make sure that you increase the Serial number every time you make a change on the file or else it won't notice the differences and won't update itself.

```
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     ns1.iskenhub.info. admin.iskenhub.info. (
                     2019062801         ; Serial
                          28800         ; Refresh
                           7200         ; Retry
                        2419200         ; Expire
                            600 )       ; Negative Cache TTL
;

iskenhub.info.    IN      NS      ns1.iskenhub.info. 
iskenhub.info.    IN      NS      ns2.iskenhub.info.  
ns1     IN      A       192.168.56.101 
ns2     IN      A       192.168.56.102
@       IN      A       34.90.10.12
www     IN      A       34.90.10.12
```

### An A record is the record that points the Domain name to the IP address.

### An NS record is to define the nameservers of the web server.

### '@' is loopback (points to the self.).


According to RFC1537 SOA(Start of Authority) Records below are recommended for a DNS server.

### YYYYMMDD (2019-09-01); Traditional Serial

### 28800; Refresh (8 hours) The time interval (in seconds) before the zone should be refreshed.
 
### 7200; Retry (2 hours) The time interval (in seconds) before a failed refresh should be retried.

### 604800; Expire (7 days) Seconds before a zone is no longer considered authoritative.

### 86400; Minimum TTL (1 day)


Then edit the reverse zone file.

`
sudo nano  /etc/bind/zones/db.34.90
`

```
; BIND reverse data file for empty rfc1918 zone
$TTL    86400
@       IN      SOA     iskenhub.info. admin.iskenhub.info. (
                           2019062801   ; Serial
                                28800   ; Refresh
                                 7200   ; Retry
                              2419200   ; Expire
                                  600 ) ; Negative Cache TTL
;
        IN      NS      ns1.iskenhub.info.
        IN      NS      ns2.iskenhub.info.
101     IN      PTR     ns1.iskenhub.info.
102     IN      PTR     ns2.iskenhub.info. 
12     IN      PTR     www.iskenhub.info. 
```

### PTR record is the Reverse record that points the IP back to the domain name.

Control the syntax errors on zone files

`
sudo named-checkconf
`

`sudo named-checkzone iskenhub.info /etc/bind/zones/db.iskenhub.info
`

The output should be like

`
zone iskenhub.info/IN: loaded serial 6
OK
`

`
sudo named-checkzone 56.168.192.in-addr.arpa /etc/bind/zones/db.34.90
`
get a similar output as before.

Restart bind

`
sudo service bind9 restart
`

To troubleshoot:

`
sudo tail -f /var/log/syslog
`

`
systemctl status bind9
`

# Slave configuration

`
sudo nano /etc/bind/named.conf.local
`

```
zone "iskenhub.info" {
        type slave;
        file "db.iskenhub.info";
        masters {192.168.56.101;};
};

zone "56.168.192.in-addr.arpa" {
        type slave;
        file "db.34.90";
        masters {192.168.56.101;};
};
```

`
dig iskenhub.info @localhost
`

You should be getting the Website address after the query.

To see the statistics of the website from an other host use

`
dig -t ns 34.90.10.12
`

`netstat -nltp` gives you the ports open on the machine. Check it for port 53.























