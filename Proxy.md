# Proxy


If we use a proxy server, internet traffic flows through the proxy server on its way to  the address we requested. Proxy server acts as a firewall to speed up requests. When we make a web request the request goes to the proxy server then proxy collects the responses and forwards us the web page.

VPNs are good for masking your activity online. In contrast, a proxy server doesn’t anonymize your activity until your data reaches the server itself.

## Why should we use a proxy server?

- An advantage of proxy is that it can cache frequent requests from its hard disk which improve user overall response time for internet surfing.

- Illegal usage of proxy can lead to anonymous surfing on the web which enables user privacy.

- Proxy can also be used to monitor traffic.

- Using proxies for web scraping, this is more for businesses, we need proxies when scraping so we just don’t get IP blocked.

## Forward and reverse proxy servers

- Forward proxies send requests of client to the web server.

- Reverse proxy is the server that sits in front of web servers and forwards client requests to those web servers.

- Forward proxies make sure no origin servers communicate directly with the specified client server.

- Reverse proxies are used to make sure no client server directly communicates with the origin server.

- Open proxy is accessible by any internet user that changes their IP address once used.

- An origin server is the server that responds to client requests. 

- DNS proxies forward DNS requests from LANs to DNS servers while caching for increased speed. 

 The only real privacy element of a proxy is that they hide our IP address. There aren’t any encryption elements like in a VPN so our traffic can be accessed. To fully stay protected in the long term, VPN is the greater solution if one can afford its processing and money costs.
 
  VPN is also inserted to the operating system so it captures all the traffic from our system instead of a proxy which only protects whatever application it is attached to(HTTP, SSL, FTP etc.)

## HAPROXY

 HAProxy, which stands for High Availability Proxy, is a popular open source software TCP/HTTP (layer 4 and layer 7) Load Balancer and proxying solution which can be run on Linux.
 
 [![ha-diagram-animated.gif](https://i.postimg.cc/g0f7g6TJ/ha-diagram-animated.gif)](https://postimg.cc/8Fmw5sDQ)

## Load Balancing Methods

- The Round Robin method, the load balancing default cycles through a list of servers in sequential order.

- Leastconn method  selects the server with the least number of connections. Not recommended for longer sessions.



# Configuring HAProxy for HTTP load balancing

first update your system

`sudo apt update
`

then install haproxy

`
sudo apt install haproxy
`

Go to the haproxy configuration file where we will be adding the servers to load balance.

` 
sudo vi /etc/haproxy/haproxy.cfg
`

Inside add the following according to your IP addresses.

```
frontend local_server
        bind 192.168.185.7:80
        mode http
        default_backend my_webs

backend my_webs
    mode http
    balance roundrobin    
    option forwardfor
    http-request set-header X-Forwarded-Port %[dst_port]
    http-request add-header X-Forwarded-Proto https if { ssl_fc }
    option httpchk HEAD / HTTP/1.1rnHost:localhost
    server web1.com 192.168.185.3:80
    server web2.com 192.168.185.4:80
```
The frontend is used to accept requests from clients by  binding the haproxy to the IP address that will be the floating(Virtual) IP which will keep redirecting to those two servers using the round robin balancing method which will be going back and forth using a round table method.

backend section is the servers that fulfill the requests.

Next see if you made any syntax errors.

You can use `listen` as an alternative to merge frontend and backend into one.

```
listen firstbalance
        bind 192.168.185.7:80
        mode http
        balance roundrobin
        option forwardfor
        server webserver1 Your-Webserver1-IP:80 
        server webserver2 Your-Webserver2-IP:80 
```

`
haproxy -c -f /etc/haproxy/haproxy.cfg
`

Restart haproxy 

`
sudo systemctl restart haproxy
`

then go to http://192.168.185.7 and keep reloading the site to see both web servers you declared.

https://www.haproxy.com/blog/the-four-essential-sections-of-an-haproxy-configuration/ for more information about the config files.

# High Availability Load Balancing using Keepalive

Keepalive is a tool used to redirect the server to the slave in case of a master server shutdown or crash.

`
sudo apt install keepalive
` 

then make it allow ipv4

`
sudo nano  /etc/sysctl.conf
`

inside add

`
net.ipv4.ip_nonlocal_bind=1
`
and run `sysctl -p`

then go to ` sudo nano /etc/keepalived/keepalived.conf`

if it's not created create it and add:

```
vrrp_instance VI_1 {
        interface enp0s3
        state MASTER
        virtual_router_id 51
        priority 101                    # 101 on master, 100 on backup
        virtual_ipaddress {
            192.168.185.10
        }
        track_script {
            chk_haproxy
        }
}
```

do the same steps for the slave haproxy server but change the priority to '100' and state to 'BACKUP'.

`
systemctl start keepalive
`

`ip addr sh enp0s3` to apply the changes and you should see the virtual address on the master server.

To test it first go to the virtual address and then stop the master server and try it again to see if it redirects to the slave server.

also you can see the change in virtual IP by looking at `systemctl status keepalived` or `ip a` to see if slave gets the Virtual IP.
