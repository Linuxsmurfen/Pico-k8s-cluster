# A Raspberry π cluster with Kubernetes
What to do when you have too many RPIs lying around and want to learn kubernetes and ansible?  
  
Build a cluster with ansible :-)

- The cluster contains of one rpi that acts as the router while the other three builds the cluster.  
- The setup of the router and cluster nodes are done via Ansible playbooks.  


## Connect everything
One RPI will be the router that is connected via wifi to the network/internet.   
All RPIs are connected to a isolated/dedicated switch that will handle the traffic between the nodes.    
```
          / --> B
wifi --> A  --> C
          \ --> D
```
| Node | Type | Hostname | Ethernet | Functions
| --- | --- | --- | --- | --- |
| A | rpi3 | router | 10.0.0.1 | Wifi, DNS, DHCP, (NFS, Loadbalancer) | 
| B | rpi3 | master | 10.0.0.50 | |
| C | rpi4 | worker4 | 10.0.0.60 | |
| D | rpi2 | worker2 | 10.0.0.61 | |


## Prepare the sd-cards
- Download the latest image  
``` wget https://downloads.raspberrypi.org/raspios_lite_armhf/images/raspios_lite_armhf-2020-12-04/2020-12-02-raspios-buster-armhf-lite.zip  ```  
``` unzip 2020-12-02-raspios-buster-armhf-lite.zip ```

- Create the sd cards  
``` sudo dd bs=4M if=2020-12-02-raspios-buster-lite-armhf.img of=/dev/sdXX status=progress conv=fsync ```

- Enable ssh  
``` touch  ../boot/ssh ```
   
- Enable wifi access only on the 'router' node  
``` ../boot/wpa_supplicant.conf ```  

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=SE

network={
  ssid="<your ssid>"
  psk="<Password for your wireless LAN>"
}
```
   
## Router node setup

Boot up the router node  

Logon to the router node as user 'pi' and install Ansible and Git
```
sudo apt install ansible git  
``` 

Download the ansible scripts  
``` git clone https://github.com/Linuxsmurfen/Pico-k8s-cluster.git  ``` 
  
Modify the MAC addresses in
```
../roles/router/files/dnsmasq.conf  
../roles/common/vars/main.yml  
```

Generate ssh keys
```
ssh-keygen  
ssh-copy-id localhost  
```

Configure the router node   
```ansible-playbook 1-routersetup.yml```

Restart the router node  
```reboot```


## Cluster node setup  
Boot the master and worker nodes  

Verify that the nodes gets an IP via DHCP  
```cat /var/lib/misc/dnsmasq.leases```

Copy the ssh keys to the cluster nodes  
```
ssh-copy-id pi@10.0.0.50
ssh-copy-id pi@10.0.0.60
ssh-copy-id pi@10.0.0.61
```  

Verify connectivity  
```ansible cluster -m ping```

Configure the nodes  
```ansible-playbook 2-clustersetup.yml``` 

Restart the nodes  
```reboot```

## Kubernetes setup
Time to setup the fun stuff.  
```git clone https://github.com/rak8s/rak8s```  

Modify the ```inventory``` file.  

Verify connectivity  
```ansible -m ping all```

Deploy the kubernetes cluster  
```ansible-playbook cluster.yml```


## Bill of materials

| Description | Link |
| --- | --- |
| 4 Raspberry PI's version 2,3,4 | https://www.raspberrypi.org/products |
| 4 sd-cards | |
| Network switch with 4 ports | For example D-Link DGS-105 |
| Power supply | For example a 60w 6 port charger |
| 4 network cables | |
| 4 power cables | |


## References & Credits
-https://www.shogan.co.uk/kubernetes/building-a-raspberry-pi-kubernetes-cluster-part-1-routing/  
-https://downey.io/blog/create-raspberry-pi-3-router-dhcp-server/  
-http://www.lpenz.org/articles/ansiblerpi/index.html  
-https://github.com/rak8s/rak8s  
