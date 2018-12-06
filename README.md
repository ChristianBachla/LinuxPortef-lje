# LinuxPortef-lje

This guide describes how to set up unprivileged containers and enabling networking between them and the host. Specifically two containers are made, where the first one, C1, runs a webserver and fetches random numbers from the second one, C2
.

lxc-net is used to make the containers. Setting up the bridge is described on this page: https://angristan.xyz/setup-network-bridge-lxc-net/

After the two unprivileged containers are created, attach the first one and install lighttpd and php5, using the following commands:

$ lxc-attach -n C1 -- apk update
$ lxc-attach -n C1 -- apk add lighttpd php5 php5-cgi php5-curl php5-fpm

The script "index.php" should be placed in /var/www/localhost/htdocs/ on container C1. It makes a http request to C2 on port 8080 using the curl command.

The script "rng.sh" should be placed in /bin/ on container C2. It fetches random numbers. The script can be served with socat, e.g.:

$ socat -v -v tcp-listen:8080,fork,reuseaddr exec:/bin/rng.sh

We can use the iptables NAT routing table to map a host's port to a container's port, with the following command:

$ sudo iptables -t nat -A PREROUTING -i <host_nic> -p tcp --dport <host_port> -j DNAT --to-destination <ct_ip>:<ct_port>

In our case host_nic=wlan0, ct_ip=10.0.3.11, host_port=80, ct_port=80.

Now the container C1's port 80 is accesible from the outside through your host's 80 port. 
