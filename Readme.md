# Test setup
The sopnode consists of various baseband and RF band devices. These devices are accessed by kubetnets pods on high performance sopnode servers. The Fabric-TNA based sopnode network called P4 network (as its switches are configurable via P4 code) facilitate this connectivity. The kubernets pods hosted on these servers connects to P4 network via kubernetes network extension and attachements. As sopnode is expected to support very high speed and low latency to support the testing of beyond 5G technology, it is required to benchmark and proliferate sopnode links and possible interconnection. This document presents a simple testsetup within sopnode kubernetes cluster to perform such performance measurements. 

![USRP Test setup](https://github.com/vinod-git/sopnode-usrp-test/blob/main/usrp-testsetup.png)
                                   
                                               Fig: USRP Test Setup

The fig. USRP Test setup depicts the typical testsetup to perform required performance measurements over the P4 network using sopnode-w2 server and USRP N300 HW device. The testsetup assumes that p4 network is operational and kubernets sopnode cluster is deployed with necessary test specific network attachements. The [usrp-test.yaml](https://github.com/vinod-git/sopnode-usrp-test) shows the example yaml description for creating a test pod. The test pod uses the docker image that is pre-loaded with necessary softwares required to perform a sample test e.g. Ettus UHD or tcpdump. 

# Tests
Following is the description of tests performed,


## N300 test
This test uses the test pod described by [usrp-test.yaml](https://github.com/vinod-git/sopnode-usrp-test) file. The test pod uses the docker image that is pre-loaded with necessary softwares required to perform a sample test for N300 USRP. It also uses the macvlan network attachment that is used by gNB pod to connect to N300 USRP. The ettus provided streaming test is used to benchmark N300 over this setup. As the Ettus streaming tests uses the ssh connection to perform test, following network modifications are needed to carry this test over P4 network connected network interfcace. 

```bash
route add -net 192.168.100.44 netmask 255.255.255.255 metric 1024 dev net1
route add -net 192.168.100.43 netmask 255.255.255.255 metric 1024 dev net1
```
