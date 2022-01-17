# Docker layer 2 extend over layer 3 network
In this documentation, we will ensure docker layer 2 conectivity over different host and network using vxlan tunnel. We will use 3 VMs for this experiment. 2 for docker host and another one will be used for gateway router. 

# Requirements:
###### Docker host1 IP: 172.16.10.100/24
###### Docker host2 IP: 172.16.20.100/24

# Scenario:
Tow docker hosts are in different network and isolated by layer 3. We will have to need bridge (br1) interface on each host and connect with docker internal network. we will create two internal network ( **10.0.1.0/24**  and **10.0.2.0/24**) on each docker host and establish layer 2 connectivity between them. We will achive this requirements by creating tunnel between these host. We will use vxlan tunneling for this experiments. So, our whole network scenario will be look like this.
