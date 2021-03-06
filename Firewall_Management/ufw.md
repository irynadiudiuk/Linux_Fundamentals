

#  IP TABLES

*Plan:*

1. About IP Tables;
2. Concepts;
3. Netfiler Framework;
4. Iptables and Connection Tracking;
5. Examples of iptables rules;
6. Resources.

-------
# ***About Iptables***

 - Linux's iptables is a common firewall built-in as a default one into many distributions.

 - Three basic concetps that any firewall is concerned about are:


```Packet``` - which is a logical container representing a flow a data; 


```Port``` - which is a numerical designation representing a particular protocol;


```Protocol``` - a language and set of rules that network devices operate by. To explain the protocol, there is a close analogy between protocols and programming languages: protocols are to communications what programming languages are to computations.

- As a typical firewall, iptables controls the ports on the network where packets can enter, pass through or exit. Ports can be opened or closed for each kind of service or kind of traffic one wishes to allow. Other ports are closed for traffic one wishes to deny.


- The netfiler framework, of which iptables is a part of, allows the system administrator to define rules for how to deal with network packets. Rules are grouped into chains — each chain is an ordered list of rules. Chains are grouped into tables — each table is associated with a different kind of packet processing. Chains are groupings of rules that govern network traffic by opening and closing ports that can be applied or bound to an interface in a particular order.

 - Below are some of the most commonly used ports. Traffic access to these ports are often configured via firewall: ![Screenshot](https://github.com/irynadiudiuk/Linux_Fundamentals/blob/master/Firewall_Management/some.png)


 - Below are the list of directories where the components of ip tables are stored:


```/proc/net/ip_tables_names``` - the list of used tables


```/proc/net/ip_tables_targets``` - the list of used actions


```/proc/net/ip_tables_matches``` - the list of used protocols


```/proc/net/nf_conntrack``` (or ```/proc/net/ip_conntrack```) - the list of opened connections and there state



- Rules are placed within a specific chain of a specific table. As each chain is called, the packet in question will be checked against each rule within the chain in order. Each rule has a matching component and an action component. The matching portion of a rule specifies the criteria that a packet must meet in order for the associated action (or "target") to be executed.

- Rules can be placed in user-defined chains in the same way that they can be placed into built-in chains. The difference is that user-defined chains can only be reached by "jumping" to them from a rule (they are not registered with a netfilter hook themselves).


# Concepts Used in Iptables

The origin of the packet determines which chain it traverses initially. 
There are five predefined chains (mapping to the five available Netfilter hooks), though a table may not have all chains. Predefined chains have a policy, for example ```DROP```, which is applied to the packet if it reaches the end of the chain. The system administrator can create as many other chains as desired. These chains have no policy; if a packet reaches the end of the chain it is returned to the chain which called it. A chain may be empty.


```PREROUTING```: Packets will enter this chain before a routing decision is made.



```INPUT```: Packet is going to be locally delivered. It does not have anything to do with processes having an opened socket; local delivery is controlled by the "local-delivery" routing table: ip route show table local.



```FORWARD```: All packets that have been routed and were not for local delivery will traverse this chain.



```OUTPUT```: Packets sent from the machine itself will be visiting this chain.



```POSTROUTING```: Routing decision has been made. Packets enter this chain just before handing them off to the hardware.


The packet continues to traverse the chain until either a rule matches the packet and decides the ultimate fate of the packet, for example by calling one of the ```ACCEPT``` or ```DROP```, or a module returning such an ultimate fate; or
a rule calls the ```RETURN``` verdict, in which case processing returns to the calling chain; or
the end of the chain is reached; traversal either continues in the parent chain (as if RETURN was used), or the base chain policy, which is an ultimate fate, is used.



# ***Netfiler Framework***


Firewalls are an important tool that can be configured to protect your servers and infrastructure.
In the Linux ecosystem, iptables is a widely used firewall tool that interfaces with the kernel's netfilter packet filtering framework. 
The iptables firewall works by interacting with the packet filtering hooks in the Linux kernel's networking stack. 
These ***kernel hooks*** are known as the ***ntfilter framework***.

There are five netfilter hooks that programs can register with. As packets progress through the stack, they will trigger the kernel modules that have registered with these hooks. The hooks that a packet will trigger depends on whether the packet is incoming or outgoing, the packet's destination, and whether the packet was dropped or rejected at a previous point.

The following hooks represent various well-defined points in the networking stack:


```NF_IP_PRE_ROUTING```: This hook will be triggered by any incoming traffic very soon after entering the network stack. This hook is processed before any routing decisions have been made regarding where to send the packet.


```NF_IP_LOCAL_IN```: This hook is triggered after an incoming packet has been routed if the packet is destined for the local system.


```NF_IP_FORWARD```: This hook is triggered after an incoming packet has been routed if the packet is to be forwarded to another host.


```NF_IP_LOCAL_OUT```: This hook is triggered by any locally created outbound traffic as soon it hits the network stack.


```NF_IP_POST_ROUTING```: This hook is triggered by any outgoing or forwarded traffic after routing has taken place and just before being put out on the wire


Within each iptables table, rules are further organized within separate "chains". While tables are defined by the general aim of the rules they hold, the built-in chains represent the netfilter hooks which trigger them. Chains basically determine when rules will be evaluated.

The names of the built-in chains mirror the names of the netfilter hooks they are associated with:


| Chain        | Netfilter hook                           | 
| :-----------:| :--------------------------------------: |
| PREROUTING   | Triggered by the NF_IP_PRE_ROUTING hook. | 
| INPUT        | Triggered by the NF_IP_LOCAL_IN hook.    | 
| FORWARD      | Triggered by the NF_IP_FORWARD hook.     | 
| OUTPUT       | Triggered by the NF_IP_LOCAL_OUT hook.   | 
|POSTROUTING   | Triggered by the NF_IP_POST_ROUTING hook.|



# ***Iptables and Connection Tracking***

Connection tracking is applied very soon after packets enter the networking stack. 
The raw table chains and some basic sanity checks are the only logic that is performed on packets prior to associating the packets with a connection.

The system checks each packet against a set of existing connections. It will update the state of the connection in its store if needed and will add new connections to the system when necessary. Packets that have been marked with the ```NOTRACK``` target in one of the raw chains will bypass the connection tracking routines.


```NEW:```When a packet arrives that is not associated with an existing connection, but is not invalid as a first packet, a new connection will be added to the system with this label. This happens for both connection-aware protocols like TCP and for connectionless protocols like UDP.


```ESTABLISHED:``` A connection is changed from NEW to ESTABLISHED when it receives a valid response in the opposite direction. For TCP connections, this means a SYN/ACK and for UDP and ICMP traffic, this means a response where source and destination of the original packet are switched.


```RELATED:``` Packets that are not part of an existing connection, but are associated with a connection already in the system are labeled RELATED. This could mean a helper connection, as is the case with FTP data transmission connections, or it could be ICMP responses to connection attempts by other protocols.


```INVALID:``` Packets can be marked INVALID if they are not associated with an existing connection and aren't appropriate for opening a new connection, if they cannot be identified, or if they aren't routable among other reasons.


```UNTRACKED:``` Packets can be marked as UNTRACKED if they've been targeted in a raw table chain to bypass tracking.


```SNAT:``` A virtual state set when the source address has been altered by NAT operations. This is used by the connection tracking system so that it knows to change the source addresses back in reply packets.


```DNAT:``` A virtual state set when the destination address has been altered by NAT operations. This is used by the connection tracking system so that it knows to change the destination address back when routing reply packets.

# ***Examples of iptables rules***


```iptables -F``` (or) ```iptables --flush``` - to flush all existing rules


```iptables -P INPUT DROP``` - sets default policy for INPUT chain


```iptables -P FORWARD DROP``` - sets default policy for FORWARD chain


```iptables -P OUTPUT DROP``` - sets default policy for OUTPUT chain


```iptables -A INPUT -i eth0 -s "$BLOCK_THIS_IP" -j DROP``` - blocks only TCP traffic on eth0 connection for specific ip-address


```iptables -A INPUT -i eth0 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT``` - allows ALL incoming ssh connections on eth0 interface.


```iptables -A INPUT -i eth0 -p tcp -m multiport --dports 22,80,443 -m state --state NEW,ESTABLISHED -j ACCEPT``` - allows all incoming SSH, HTTP and HTTPS traffic.

The screenshot below shows the most commonly used switches:
![Screenshot](https://github.com/irynadiudiuk/Linux_Fundamentals/blob/master/Firewall_Management/switches.PNG)

____________________________________________
 
 # ***Resources***
 
1. https://www.digitalocean.com/community/tutorials/a-deep-dive-into-iptables-and-netfilter-architecture;
2. https://www.youtube.com/watch?v=1PsTYAd6MiI;
3. https://en.wikibooks.org/wiki/Communication_Networks/IP_Tables
4. http://www.thegeekstuff.com/2011/06/iptables-rules-examples
5. https://www.youtube.com/watch?v=1PsTYAd6MiI&t=307s
6. http://www.k-max.name/linux/netfilter-iptables-v-linux/

