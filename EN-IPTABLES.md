# What is IPTABLES?
Iptables is a generic table structure that defines rules and commands as part of the netfilter framework that facilitates Network Address Translation (NAT), packet filtering, and packet mangling in Linux. NAT is the process of converting an Internet Protocol address (IP address) into another IP address. Packet filtering is the process of passing or blocking packets at a network interface based on source and destination addresses, ports, or protocols. Packet mangling is the ability to alter or modify packets before and/or after routing.

### Tables:

The ***raw*** table is used only for configuring packets so that they are exempt from connection tracking.

The ***filter*** table is the default table, and is where all the actions typically associated with a firewall take place.

The ***nat*** table is used for network address translation (e.g. port forwarding).

The ***mangle*** table is used for specialized packet alterations.

The ***security*** table is used for ***M***andatory ***A***ccess ***C***ontrol* networking rules (e.g. SELinux)

In most common use cases you will only use *filter*, *nat* and *mangle*

### Chains:
Tables consist of chains, which are lists of rules which are followed in order.
The *filter* table, contains three built-in chains: *INPUT*, *OUTPUT* and *FORWARD*

The nat table includes *PREROUTING*, *POSTROUTING*, y *OUTPUT*.
Chains do have a default policy, which is generally set to *ACCEPT*, but can be reset to *DROP*, if you want to be sure that nothing slips through your ruleset.

`INPUT`: This chain is used to control the behavior for incoming connections.
`OUTPUT`: This chain is used for outgoing connections.
`FORWARD`: This chain is used for incoming connections that aren’t actually being delivered locally. 
`PREROUTING`: This chain is used for altering packets as soon as they come in.
`POSTROUTING`: This chain is used for altering packets as they are about to go out.
`OUTPUT`: This chain is used for locally-generated packets.

### Reglas:
Packet filtering is based on rules, which are specified by multiple matches (conditions the packet must satisfy so that the rule can be applied), and one target (action taken when the packet matches all conditions). The typical things a rule might match on are what interface the packet came in on, what type of packet it is, or the destination port of the packet.
Targets are specified using the *-j* o *--jump*. Built-in targets are *ACCEPT*, *DROP*, *QUEUE* , *RETURN* , *REJECT* o *LOG* .

Once you understand this...

# How we can improve the security of our servers with IPTABLES?

A good practice, its flush all our rules.
For this, we need open the terminal and write:

`iptables -F`
`iptables -P FORWARD ACCEPT`
`iptables -P OUTPUT ACCEPT`
`iptables -P INPUT ACCEPT`

`-F` - flush --> This is equivalent to deleting all the rules one by one.
`-P` - policy → Set the policy for the chain to the given target.

Then, we ll set as default drop everything and start open the ports we ll use.

`iptables -P INPUT DROP`
`iptables -P FORWARD DROP`
`iptables -P OUTPUT DROP`

With this commands, we ll not have internet, so we must start creating permissive rules.

### Allow SSH port
iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT

**RECOMMENDATION** 
One of the benefits of change SSH Port is avoid the typical port scan and the script kiddies. Most hackers search Port 22 for SSH. For that reason its a good practice change it. And if you can, make a Honeypot in port 22. Also you can allow SSH only from a specific network.

### Allow Incoming HTTP and HTTPS (Apache)
`iptables -A INPUT -m state --state NEW -p tcp --dport 80 -j ACCEPT`
`iptables -A INPUT -m state --state NEW -p tcp --dport 443 -j ACCEPT`

### Allow IMAP, SMTP y POP3 ports (In case we have email server)
`iptables -A INPUT -m state --state NEW -p tcp --dport 143 -j ACCEPT`
`iptables -A INPUT -m state --state NEW -p tcp --dport 25 -j ACCEPT`
`iptables -A INPUT -m state --state NEW -p tcp --dport 110 -j ACCEPT`

### Allow DNS Server
`iptables -A INPUT -m state --state NEW -p udp --dport 53 -j ACCEPT
iptables -A INPUT -m state --state NEW -p tcp --dport 53 -j ACCEPT`

