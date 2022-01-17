# Docker layer 2 network extend over layer 3 isolation
In this documentation, we will ensure docker layer 2 conectivity over different host and network using vxlan tunnel. We will use 3 VMs for this experiment. 2 for docker host and another one will be used for gateway router. 

# Requirements:
###### Docker host1 IP: 172.16.10.100/24
###### Docker host2 IP: 172.16.20.100/24

# Scenario:
Tow docker hosts are in different network and isolated by layer 3. We will have to need bridge (br1) interface on each host and connect with docker internal network. In that case Open vSwitch (**OVS**) will be used. we will create two internal network ( **10.0.1.0/24**  and **10.0.2.0/24**) on each docker host and establish layer 2 connectivity between them. We will achive this requirements by creating tunnel between these host. We will use vxlan tunneling for this experiments. So, our whole network scenario will be look like this.

![2](https://user-images.githubusercontent.com/22352861/149739304-68005da3-5191-432e-a88c-9a27e61d7814.PNG)

# specifications:
###### net1: 10.0.1.0/24
###### net2: 10.0.1.0/24

# Configuration:
### Docker host1:
First we will launch two dockers **doc1** and **doc2** from host1.
  
  docker run -di --name doc1 debian\
  docker run -di --name doc2 debian

Now, create a bridge interface named **br1** using ovs utility.

  sudo ovs-vsctl add-br br1
  
Then create two ovs internal port (**net1** and **net2** under bridge **br1**.

  sudo ovs-vsctl add-port br1 net1 -- set interface net1 type=internal\
  sudo ovs-vsctl add-port br1 net2 -- set interface net2 type=internal

we will now configure IP on these two ports otherwise host machine doesn't have the route to reach **net1** and **net2** network.

  sudo ifconfig net1 10.0.1.1 netmask 255.255.255.0 up\
  sudo ifconfig net2 10.0.2.1 netmask 255.255.255.0 up
  
Now we can associated rote in host machine route table.

  


    
  
 
