<h1 style="font-size: 3em;">OAI5G</h1>

# Project Documentation

## Report  
[Open Report on Overleaf](https://www.overleaf.com/read/hmsydpyhcjzb#a70a09)

## Video Link of PPT  
[Watch Video Presentation](https://drive.google.com/file/d/1BqIHjdNuTGePsX9RhV1SA8k-AXSVgxw5/view)

Open Air Interface (OAI) Startup: (Installation and Configuration of Radio Access Network)


*   [OAI Roadmap](#OAI)

	*	[Kernal Setup and Intel Driver Configuration](#kernal)
	*	[E-UTRAN S.W. Installation](#S.W.)
	*	[Testing Radio Access Network Connection](#RAN)
	
*   [OAI Gitlab](#REF)


<h2 id="OAI">OAI Roadmap</h2>

<h3 id="kernal">Kernal and OS Requirements</h3>

First, we need to get a low latency kernal
- for Ubuntu 24.04
```
sudo apt-get install linux-image-lowlatency linux-headers-lowlatency
```
- After Reboot You can check the installed kernal by typing
```bash
uname -a
``` 
#### OAI Power Management
OAI requires to Remove all power management features in the BIOS (sleep states, in particular C-states) and CPU frequency scaling (Intel SpeedStep) so OAI can run with the maximum CPU clock 100% in performance mode and avoid  any scaling, also make sure to disable the p-state driver 

Add GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_pstate=disable" to /etc/default/grub
```bash
nano /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_pstate=disable"
```
Next we update the Governor to performance

```bash
sudo apt-get install cpufrequtils
sudo echo GOVERNOR="performance"  >  /etc/default/cpufrequtils
sudo update-rc.d ondemand disable //to maintain the setting across system reboot
sudo /etc/init.d/cpufrequtils restart 
```
###### Checking the CPU cores
```bash
sudo cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```

<h3 id="S.W.">E-UTRAN S.W. Installation</h3>

Checkout RAN repository (eNB RAN + UE RAN):
- for Ubuntu 24.04 Get the develop branch (up to date features)
```bash
git clone -b develop https://gitlab.eurecom.fr/oai/openairinterface5g.git
```

```bash
cd openairinterface5g
```

###### It sets the correct environment variables. 
```bash
source oaienv
```
###### package installation + USRP Driver installation :
```bash
./build_oai -I
./build_oai -I -w USRP
```
###### Build OpenAirInterface eNB and UE without the EPC (without S1, nos1) 
```bash
./build_oai -w USRP --eNB --UE --noS1 -x
```
This command works only with master branch, it will not work with the develop branch because of the seperation of eNB and UE, so to compile only pass one argument at a time, UE either eNB.
```bash
./build_oai -w USRP --eNB --noS1 -x
```
<p align="center">
  <img src="https://github.com/astro7x/oai5g/blob/master/img/RAN_noS1.png?raw=true"/>
</p>

<h3 id="RAN">Testing Radio Access Network Connection</h3>

### Running eNB

1st: Setup eNB IP
```bash
cd ~/openairinterface5g/
source oaienv
source ./cmake_targets/tools/init_nas_nos1 eNB
```
2nd: Check eNB IP

```bash
ifconfig 
```

```
oai0      Link encap:AMPR NET/ROM  HWaddr C2-C6-7B-07-22-08 
          inet addr:10.0.1.1  Bcast:10.0.1.255  Mask:255.255.255.0
          UP BROADCAST RUNNING NOARP MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:100 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

```


```bash
cd cmake_targets
sudo -E ./lte_noS1_build_oai/build/lte-softmodem-nos1 -d -O $OPENAIR_TARGETS/PROJECTS/GENERIC-LTE-EPC/CONF/enb.band7.tm1.usrpb210.conf 2>&1 | tee ENB.log
```
<p align="center">
  <img src="https://github.com/astro7x/oai5g/blob/master/img/eNB0.png?raw=true"/>
</p>



### Running UE

1st: Setup UE IP
```bash
cd ~/openairinterface5g/
source ./targets/bin/init_nas_nos1 UE
```

2nd: Check UE IP

```bash
ifconfig 
```

```
oai0      Link encap:AMPR NET/ROM  HWaddr 71-7C-CA-24-25-0B
          inet addr:10.0.1.9  Bcast:10.0.1.255  Mask:255.255.255.0
          UP BROADCAST RUNNING NOARP MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:100 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

```bash
cd cmake_targets
sudo -E ./lte_noS1_build_oai/build/lte-softmodem-nos1  -U -C2660000000 -r25 --ue-scan-carrier --ue-txgain 90 --ue-rxgain 115 -d >&1 | tee UE.log
```

<p align="center">
  <img src="https://github.com/astro7x/oai5g/blob/master/img/UE0.png?raw=true"/>
</p>

<h3 id="REF">OAI Gitlab</h3>

OpenAirInterface is under OpenAirInterface Software Alliance license.
Refere to the main source code at GitLab

#### Refere to https://gitlab.eurecom.fr/oai/openairinterface5g/tree/master for the full source code

