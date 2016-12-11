# Working LOG --- enable acl functionality
## updated Nov.30/2016

___Updating Aim:___ realizing and test each of ACL works correctly
### Updated items
modified simple_router_acl_v1.p4. Enabling and tested following acl functionality. Amoung matching, actions toward packets are either being droped or conducted no operation.

1. ip_acl 
2. tcp_acl 
3. prot_acl 

### Testing ACL
To test the each acl functionality is correctly working, basically after compile the P4 file, invoke p4 switch in mininet through command
```
./run_demo.bash
```
Open another terminal, under the same directory run:
```
./run_add_demo_entries.bash
```
This will insert basic routing rules and designed ACL rules to P4 switch.
Next, in the mininet, fire xterm on both h1 and h2 by:
```
xterm h1 h2
```
Choose iperf generated tcp and udp packets to test our ACL. 
Each of individual ACL functionality test requires changes to **run_add_demo_entries.bash** file corresbonding lines. While changing, comment out ACLs that are not needed.

#### 1. ip_acl: 
This part test to drop all packet from h2(10.0.1.10) (or certain IP range). 
Before starting, we want to monitor the traffic inbound and outbound on switch. Open another terminal and type in:
```shell
sudo wireshark
```
And choose to listen on port s1 and s2.
In xterm h1 fire:
```
iperf -s
```
In xterm h2 fire:
```
iperf -c 10.0.0.10
```
Note the switch will block all the packets originated from h2. Meaning no tcp 3-way hand shake will be successful. And thus it would be the consequence as h2 keep sending ACK to h1 but with no response. Here is what I got in this experiement: 
[![wireshark listen with packets originated from h2's IP droped.](https://s14.postimg.org/yxpinbdip/ip_acl.jpg)](https://postimg.org/image/e0tainfhp/)
Upon trying to set up connection, packets are blocked and keep retransmit. 
#### 2. tcp_acl:
This part test to drop packet from or desting to certain port number. Similar to the above experiement, send tcp packet from h2 to h1 with dst port as iperf default server port 5001. Here is what I got in this experiement:

First. Test the connectivty amoung the link by run:
```
pingall
```
and get result:

[![pingall to test connectivity](https://s12.postimg.org/g8h1phtst/20161130205719.jpg)](https://postimg.org/image/z0swt2q6x/)

Second. Generate tcp traffic with certain dst port number and fire wireshark, Then I got this:
[![wireshark listen with packets dst to port 5001 droped](https://s14.postimg.org/yqkrgopip/20161130234920.png)](https://postimg.org/image/j53fwqdkd/)
How ever, our experiement experiencing a problem here: ** We have to change the control part of P4 file and to comments out not used ACL ** The possible reason for that is table dependency are not clear and here we just used simple sequential apply table method. 

#### 3. prot_acl
To ensure today's work completeness, we here still going to conduct protocal ACL test individually. 
However now I successfully block protocol number 6.
For tested UDP, according to UDP properties, send UDP frame from h2 to h1, listen on port s1. sees nothing.

### To wrap up
Partially realized ACL individually and tested.
1. others
2. apply tables dependency
3. topo scale up

## updated Dec.1/2016
___Update Aim:___ realizing udp parsing, table dependency. Try to change the topo.

### UDP parsing:
Till now all individual test succeed. We then will move on to find out why apply table all at once at control ingress does not work.


## updated Dec.10 /2016
___update Aim:___ 

1. story of ACL, why we need ACL and some simple experiement design regarding it. 
2. change topo and try add more host and default rules to the whole mininet environment
3. try chained tables
4. slices 5-10

### Recent work: below are not written here in this log file.

1. successed adding entry to ternary match type.
2. successed bring up bmv2 in mininet as well as accessing to CLI all at the same time. Accessing the table
3. successed communicating through thrift server on designed port. 
4. successed see through parser and control, match-action table logic.

### Today's work
#### 1. What an ACL actually do?
> 1. standard ACL only check the source address, either to block traffic from certain network, or allow access. Standard ACL should be as close as destination. 

## Update Dec.11 /2016
___Update Aim:___ write ACL rules, test them. slices

### Topology as written. 
Rules to realize in priority order:

1. Block all traffic from IP: H2 [IP src range block]
2. allow tcp connection from h1 to h3 [protocal] / allow all ping action.
3. allow udp connection from h1 to h2 through port 5001 [dstport #]

### Basic ping test:
In this ping test, the priority of acl to allow all ping action is lower then block H2 originated packets. So all ping direct to or from H2 are droped.
```
./reset_mininet.sh
./run_demo.bash
./run_cli.bash -c localhost:22222 < command_basic_routing.txt
>pingall # in miminet
./run_cli.bash -c localhost:22222 < command_acl.txt
>pingall # also in mininet
```

### Basic TCP test:
```
>xterm h1 h3 # in mininet, following should work in reversed order.
>> root@Node:h3# iperf -s # in h3 xterm terminal
>> root@Node:h1# iperf -c 10.0.2.10 # in h2 xterm terminal
```
### Basic port number test:
Due to the acl rule of drop all packet originated from h2, we then choose to see if open up a udp server with certain port will receive any packet or not.
```
>xterm h1 h2 # in mininet
>> root@Node:h2# iperf -s -u # in h2 xterm terminal
>> root@Node:h1# iperf -c 10.0.1.10 -u # in h1 xterm terminal
```
