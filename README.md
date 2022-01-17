# Docker layer 2 network extend over layer 3 isolation
In this documentation, we will ensure docker layer 2 conectivity over different host and network using vxlan tunnel. We will use 3 VMs for this experiment. 2 for docker host and another one will be used for gateway router. 

# Requirements:
###### Docker host1 IP: 172.16.10.100/24
###### Docker host2 IP: 172.16.20.100/24

# Scenario:
Tow docker hosts are in different network and isolated by layer 3. We will have to need bridge (br1) interface on each host and connect with docker internal network. In that case Open vSwitch (**OVS**) will be used. we will create two internal network ( **10.0.1.0/24**  and **10.0.2.0/24**) on each docker host and establish layer 2 connectivity between them. We will achive this requirements by creating tunnel between these host. We will use vxlan tunneling for this experiments. So, our whole network scenario will be look like this.

![2](https://user-images.githubusercontent.com/22352861/149739304-68005da3-5191-432e-a88c-9a27e61d7814.PNG)

# Specifications:
###### net1: 10.0.1.0/24
###### net2: 10.0.1.0/24

# Configuration:
### Docker host1:
First we will launch two dockers **doc1** and **doc2** from host1.
  
  docker run -di --name doc1 debian\
  docker run -di --name doc2 debian

Now, create a bridge interface named **br1** using ovs utility.

  sudo ovs-vsctl add-br br1
  
Then create two ovs internal port **net1** and **net2** under bridge **br1**.

  sudo ovs-vsctl add-port br1 net1 -- set interface net1 type=internal\
  sudo ovs-vsctl add-port br1 net2 -- set interface net2 type=internal

we will now configure IP on these two ports otherwise host machine doesn't have the route to reach **net1** and **net2** network.

  sudo ifconfig net1 10.0.1.1 netmask 255.255.255.0 up\
  sudo ifconfig net2 10.0.2.1 netmask 255.255.255.0 up
  
Now we can associated rote in host machine route table.

![3](https://user-images.githubusercontent.com/22352861/149746971-1fe0bc28-5580-4fa1-8ac1-d5928b0d1cbb.PNG)

Now the exciting part! We will create a vxlan port to **br1** bridge named **vtep**.

  sudo ovs-vsctl add-port br1 vtep -- set interface vtep type=vxlan option:remote_ip=172.16.20.100 option:key=flow ofport_request=10
  
We can check **br1** port status by below command.

  sudo ovs-vsctl show
  
Above we have created **net1** and **net2** interface. It's time to connect these network with our containers. we will attach additional interface with IP on previously created containers. **doc1** will have interface from **net1 (10.0.1.0/24)** and **doc2** will have interface belongs to **net2 (10.0.2.0/24)** network. For this configuration apply below commands.

  sudo ovs-docker add-port br1 eth1 doc1 --ipaddress=10.0.1.10/24 \
  sudo ovs-docker add-port br1 eth1 doc2 --ipaddress=10.0.2.10/24 

### Docker host2:

At first we will launch two dockers **doc3** and **doc4** from host2.
  
  docker run -di --name doc3 debian\
  docker run -di --name doc4 debian

Create a bridge interface named **br1** using ovs utility.

  sudo ovs-vsctl add-br br1

Now creating vxlan port under **br1** bridge named **vtep**. After this vxlan tunnel will be formed with host1.

  sudo ovs-vsctl add-port br1 vtep -- set interface vtep type=vxlan option:remote_ip=172.16.10.100 option:key=flow ofport_request=10
  
Then, configuring IP on containers eth1 interface from same **net1** and **net2** network.

  sudo ovs-docker add-port br1 eth1 doc3 --ipaddress=10.0.1.20/24\
  sudo ovs-docker add-port br1 eth1 doc4 --ipaddress=10.0.2.20/24
  
Our configuration is over. we can check now.

# verifying:
Now, we will check our configuration. First ping **doc3** from **doc1** container. They are in **net1**.

![44](https://user-images.githubusercontent.com/22352861/149752929-0d044edb-130b-4c80-80ae-e3040a2fb98a.PNG)

Now ping **doc4** from **doc2** container.

![55](https://user-images.githubusercontent.com/22352861/149752979-26961764-55bc-43c6-8e63-d2db425c118c.PNG)

We can see ping is succeeding. we will get similar result from host2 container also because a distribube layer 2 network is created between these two docker host. 


    
  
 
