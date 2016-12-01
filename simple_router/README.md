## Working LOG --- enable acl functionality
updated Nov.30/2016

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

1. Test the connectivty amoung the link by run:
```
pingall
```
and get result:

[![pingall to test connectivity](https://s12.postimg.org/g8h1phtst/20161130205719.jpg)](https://postimg.org/image/z0swt2q6x/)

2. Generate tcp traffic with certain dst port number and fire wireshark, Then I got this:
[![wireshark listen with packets dst to port 5001 droped](https://s14.postimg.org/yqkrgopip/20161130234920.png)](https://postimg.org/image/j53fwqdkd/)
How ever, our experiement experiencing a problem here: ** We have to change the control part of P4 file and to comments out not used ACL ** The possible reason for that is table dependency are not clear and here we just used simple sequential apply table method. 

#### 3. prot_acl
To ensure today's work completeness, we here still going to conduct protocal ACL test individually. 
However now I successfully block protocol number 6, but not number 17. werid.

### To wrap up
Partially realized ACL individually and tested.
1. prot_acl udp and others
2. apply tables dependency
3. topo scale up
