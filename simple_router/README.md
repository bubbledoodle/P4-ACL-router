## working log --- enable acl functionality
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
Choose iperf to test our ACL. 

#### 1. ip_acl: 
inserted as rule to drop all packet from h2(10.0.1.10). 
Before starting, we want to monitor the traffic inbound and outbound on switch. Open another terminal and type in:
```
sudo wireshark
```
And choose to listen on port s1 and s2.
In xterm h1 fire:
```
iperf -s
```
In xterm h2 fire:
```
iperf -c 10.0.0.1
```
Note the switch will block all the packets originated from h2. 

#### 2. tcp_acl:
