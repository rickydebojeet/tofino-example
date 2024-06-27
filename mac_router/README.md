# Load driver if not loaded
```
cd $SDE/install/lib/modules
sudo insmod bf_kpkt.ko
```


# Connection and configuration
Consider Server A, Server B, and Server C are connected with tofino switch at port 49, 50, and 52
Server A IPv4 = 192.168.150.10/24, MAC: 3c:fd:fe:9e:7b:5d
Server B IPV4 = 192.168.150.20/24, MAC: 3c:fd:fe:9e:7e:34
Server C IPV4 = 192.168.150.30/24, MAC: 3c:fd:fe:9e:7e:35

# Compilation
```
cmake $SDE/p4studio/ -DCMAKE_INSTALL_PREFIX=$SDE/install -DCMAKE_MODULE_PATH=$SDE/cmake -DP4_NAME=mac_router -DP4_PATH=<full path of mac_router.p4>
make
sudo make install
```

# Loading the code
Please run the following commands onwards in a 'screen' session. This terminal need to remain open throughout the duration of using Tofino switch. If a screen session is not used a disconnected SSH will stop the Tofino.

```
$SDE/run_switchd.sh -p mac_router
```

# Set Up the ports for 40 Gbps NIC
```
ucli
pm
port-add 49/- 40G NONE
an-set 49/- 2
port-enb 49/-
port-add 50/- 40G NONE
an-set 50/- 2
port-enb 50/-
port-add 52/- 40G NONE
an-set 52/- 1
port-enb 52/-
show
# Wait for 10 second
show
exit
```

Check the "OPR" field from the output. It should show "UP" for both the ports if you are using Kernel network stack. Once dpdk driver is bound to the interfaces, the "OPR" field shows "UP" only after running the application. Please *DO NOT* proceed further until these are showing *"UP"*. Please not down the value in D_P field correspondig to each port and use that value while installing the rules, for e.g., for PORT 49 and 50 D_P is 28 and 44 respectively.

# Installing rules
### Start bf runtime
```
bfrt_python
```
From here onwards it is a python shell.
*NOTE:* Replace the IP, MAC and D_P according to your set up.

### ARP Rules
```
bfrt.mac_router.pipe.SwitchIngress.t_arp.add_with_a_arp("192.168.150.10", "3c:fd:fe:9e:7b:5d")
bfrt.mac_router.pipe.SwitchIngress.t_arp.add_with_a_arp("192.168.150.20", "3c:fd:fe:9e:7e:34")
bfrt.mac_router.pipe.SwitchIngress.t_arp.add_with_a_arp("192.168.150.30", "3c:fd:fe:9e:7e:35")

```

### MAC Rules
```
bfrt.mac_router.pipe.SwitchIngress.t_mac_forward.add_with_a_mac_forward("3c:fd:fe:9e:7b:5d", 28)
bfrt.mac_router.pipe.SwitchIngress.t_mac_forward.add_with_a_mac_forward("3c:fd:fe:9e:7e:34", 60)
bfrt.mac_router.pipe.SwitchIngress.t_mac_forward.add_with_a_mac_forward("3c:fd:fe:9e:7e:35", 44)

```

Server A and B should be able to ping each other at this point
