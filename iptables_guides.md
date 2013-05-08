Linux IPTABLES Rules Example
=================================================================
> The following are simple iptables firewalls for Linux. Linux comes with a host based firewall called Netfilter(iptables)

#Understanding Firewall

###There are total 4 chains:
*   INPUT - The default chain is used for packets addressed to the system. Use this to open or close incoming ports (such as 80,25, and 110 etc) and ip addresses / subnet (such as 202.54.1.20/29).
*	OUTPUT - The default chain is used when packets are generating from the system. Use this open or close outgoing ports and ip addresses / subnets.
*	FORWARD - The default chains is used when packets send through another interface. Usually used when you setup Linux as router. For example, eth0 connected to ADSL/Cable modem and eth1 is connected to local LAN. Use FORWARD chain to send and receive traffic from LAN to the Internet.
*	RH-Firewall-1-INPUT - This is a user-defined custom chain. It is used by the INPUT, OUTPUT and FORWARD chains.

###Packet Matching Rules

*	Each packet starts at the first rule in the chain .
*	A packet proceeds until it matches a rule.
*	If a match found, then control will jump to the specified target (such as REJECT, ACCEPT, DROP).

###Target Meanings

*	The target `ACCEPT` means allow packet.
*	The target `REJECT` means to drop the packet and send an error message to remote host.
*	The target `DROP` means drop the packet and do not send an error message to remote host or sending host.

##Configuring IPtables on ubuntu server
Whenever you setup any internet facing server firewalling is very important. so you can have a more secure installation. To start with, we’re going to have three ports open: ssh, http and https.
> Note: If for some reason iptables is not installed, run the command
```
	apt-get install iptables
```
Create two files,` /etc/iptables.test.rules` and `/etc/iptables.up.rules`. The first is a `temporary` (test) set of rules and the second the `permanent` set of rules (this is the one iptables will use when starting up after a reboot for example).
> Note: that we are logged in as the root user. This is the only time we will log in as the root user. As such, if you are completing this step at a later date using the admin user, you will need to put a `sudo` in front of the commands.
Now let’s see what’s running at the moment:
```    
	iptables -L
```
Look at [this example iptables configuration file](/data/iptables_configuration_example.txt).
You can change and add ports as you see fit. Don’t forget to change the port number `30000` to whatever you specified in your sshd_config.
Defined your rules? Then lets apply those rules to server:
```
    iptables-restore < /etc/iptables.test.rules
```
Let’s see if there is any difference:
```
    iptables -L
```
Once done with the rules, it’s time to save our rules permanently:
```
    iptables-save > /etc/iptables.up.rules
```

###1: Displaying the Status of Your Firewall
Type the following command as root:
```
# iptables -L -n -v
```
Sample outputs:
```
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
``` 
Above output indicates that the firewall is not active. The following sample shows an active firewall:
```
# iptables -L -n -v
```
Sample outputs:
```
Chain INPUT (policy ACCEPT 42 packets, 16614 bytes)
 pkts bytes target     prot opt in     out     source               destination         
 161K   12M fail2ban-ssh  tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            multiport dports 22

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 46 packets, 9977 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain fail2ban-ssh (1 references)
 pkts bytes target     prot opt in     out     source               destination         
 157K   11M RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0           
```

Where,

*	-L : List rules.
*	-v : Display detailed information. This option makes the list command show the interface name, the rule options, and the TOS masks. The packet and byte counters are also listed, with the suffix 'K', 'M' or 'G' for 1000, 1,000,000 and 1,000,000,000 multipliers respectively.
*	-n : Display IP address and port in numeric format. Do not use DNS to resolve names. This will speed up listing.
####1.1: To inspect firewall with line numbers, enter:
```
# iptables -n -L -v --line-numbers
```
Sample outputs:
```
Chain INPUT (policy DROP)
num  target     prot opt source               destination
1    DROP       all  --  0.0.0.0/0            0.0.0.0/0           state INVALID
2    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           state RELATED,ESTABLISHED
3    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0
4    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0
Chain FORWARD (policy DROP)
num  target     prot opt source               destination
1    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0
2    DROP       all  --  0.0.0.0/0            0.0.0.0/0           state INVALID
3    TCPMSS     tcp  --  0.0.0.0/0            0.0.0.0/0           tcp flags:0x06/0x02 TCPMSS clamp to PMTU
4    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           state RELATED,ESTABLISHED
5    wanin      all  --  0.0.0.0/0            0.0.0.0/0
6    wanout     all  --  0.0.0.0/0            0.0.0.0/0
7    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0
Chain OUTPUT (policy ACCEPT)
num  target     prot opt source               destination
Chain wanin (1 references)
num  target     prot opt source               destination
Chain wanout (1 references)
num  target     prot opt source               destination
```
> You can use line numbers to delete or insert new rules into the firewall.

####1.2: To display INPUT or OUTPUT chain rules, enter:
```
# iptables -L INPUT -n -v
# iptables -L OUTPUT -n -v --line-numbers
```

###2: Block everything firewall using bash
> This blocks everything. You will only be able to access the machine from the console. Don't do this if you are working remotely because your connection will instantly be dropped. Another way to do this would be to disable the network interface. The advantage of blocking everything with iptables instead of shutting down a network interface is that this leaves the kernel network layer still running. Applications will not complain about the network being unavailable. This also blocks all network interfaces at once, so if you have a machine with multiple interfaces this will take care of them all.
```
#!/bin/sh
iptables -P INPUT ACCEPT
iptables -P OUTPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -F
iptables -X
iptables -P INPUT DROP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP
```
###3: Allow everything firewall using bash
> This opens up everything. It's the exact opposite of [Block everything](#2-Block-everything-firewall). The firewall is still technically running, but every packet is allowed through. This is the safe way to open the firewall without accidentally locking yourself out.
```
#!/bin/sh
iptables -P INPUT ACCEPT
iptables -P OUTPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -F
iptables -X
```
###4:Ban -- block an annoying machine
> This blocks a specific IP address from reaching your server. This is useful if you are getting annoying traffic from another machine and you want to get rid of them. Replace 255.255.255.255 with the IP address you want to drop.
```
iptables -I INPUT -j DROP -s 255.255.255.255
```
> This un-blocks a specific IP address from your server. This is useful if you are get blocked from your server. Replace 255.255.255.255 with the IP address you want to un-ban.
```
iptables -D INPUT -j DROP -s 255.255.255.255
```
###5: Minimal emergency firewall using bash
> I use this to shut down everything except SSH port 22. This is my panic script. If something seems suspicious then I use this script to put a machine into as safe a state as possible while still allowing remote SSH connections.
```
#!/bin/sh
# Minimal emergency firewall (block everything except SSH).
iptables -P INPUT ACCEPT
iptables -P OUTPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -F
iptables -X
iptables -A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
iptables -P INPUT DROP
iptables -A OUTPUT -p tcp -m tcp --sport 22 -j ACCEPT
iptables -P OUTPUT DROP
```
###5: Stop / Start / Restart the Firewall
> You can save existing firewall rules as follows:
```
$ sudo iptables-save > firewall.rules
```
> Type the following commands to stop firewall:
$ sudo iptables -X
$ sudo iptables -t nat -F
$ sudo iptables -t nat -X
$ sudo iptables -t mangle -F
$ sudo iptables -t mangle -X
$ sudo iptables -P INPUT ACCEPT
$ sudo iptables -P FORWARD ACCEPT
$ sudo iptables -P OUTPUT ACCEPT

> If you are using ubuntu Linux, enter:
```
$ sudo ufw disable
$ sudo ufw enable
$ sudo ufw disable
```
You can use the iptables command itself to stop the firewall and delete all rules:
```
# iptables -F
# iptables -X
# iptables -t nat -F
# iptables -t nat -X
# iptables -t mangle -F
# iptables -t mangle -X
# iptables -P INPUT ACCEPT
# iptables -P OUTPUT ACCEPT
# iptables -P FORWARD ACCEPT
```
Where,

-F : Deleting (flushing) all the rules.
-X : Delete chain.
-t table_name : Select table (called nat or mangle) and delete/flush rules.
-P : Set the default policy (such as DROP, REJECT, or ACCEPT). 