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
Following is the ettus defined streaming test for N300.
```bash
<test executable path>/benchmark_rate --args "type=n3xx,mgmt_addr=192.168.100.43,addr=192.168.100.44,master_clock_rate=125e6" --duration 60 --channels "0" --rx_rate 3.84e6  --rx_subdev "A:0" --tx_rate 3.84e6  --tx_subdev "A:0
```
This is a test with example inputs provided by Ettus. Note that this test can only be run with server hosting kubernetes test pod and N300 device are on the same network i.e. p4 network. The output for the above mentioned test is as follows,

```bash
Using Device: Single USRP:
  Device: N300-Series Device
  Mboard 0: n300
  RX Channel: 0
    RX DSP: 0
    RX Dboard: A
    RX Subdev: Magnesium
  TX Channel: 0
    TX DSP: 0
    TX Dboard: A
    TX Subdev: Magnesium

[00:00:02.982997935] Setting device timestamp to 0...
Setting TX spp to 365
[00:00:02.987762818] Testing receive rate 3.840000 Msps on 1 channels
[00:00:02.990513855] Testing transmit rate 3.840000 Msps on 1 channels
[00:01:03.258641779] Benchmark complete.


Benchmark rate summary:
  Num received samples:     231352797
  Num dropped samples:      0
  Num overruns detected:    0
  Num transmitted samples:  231406350
  Num sequence errors (Tx): 0
  Num sequence errors (Rx): 0
  Num underruns detected:   0
  Num late commands:        0
  Num timeouts (Tx):        0
  Num timeouts (Rx):        0
```
