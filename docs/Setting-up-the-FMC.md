To enable testing and commissioning of the timing system, a timing FPGA mezzanine card(FMC) was designed and fabricated.  The FMC has SFP, Ethernet,  and  U.FL  connectors,  and  can  act  as  either  as  a  timing  master  or endpoint, depending on the firmware uploaded onto the carrier board FPGA. The carrier board can be either an Enclustra PX3/AX3 or a Digilent Nexsys Video card. There are different firmware images produced depending on which baseboard you have. 

The following instructions will allow you to set up and control the timing FMC. 

## Pre-requisites:

### Hardware: 

* Timing FMC hosted on an Enclustra Mars PM3  base  board  with  an  Enclustra  Mars  AX3  FPGA  module or Digilent Nexsys Video card
* JTAG  cable
* Linux machine (for control and firmware uploads) running CentOS 8 (though CentOS 7 still currently supported)


### Software: 

The DUNE timing butler (dtsbutler) is a python-based command line tool which provides the functionality to control the timing FMC.
The dtsbutler along with the timing software backend is part of the DUNE DAQ installation. In terms of which DAQ release to use, the following
twiki page lists the most recent tags; in general the most recent tag here is the one to use. The release will begin with the text 'dunedaq-vX.X.X'.

https://github.com/DUNE-DAQ/daq-release/tags

To install this release you can find the instructions listed here. 

https://github.com/DUNE-DAQ/daqconf/wiki




!!! warning
	You will need access to CVMFS to follow the above instructions. 

### Firmware 

Download the firmware bit file for the FMC which will allow the FMC to behave as a master timing unit - remember to download the file which corresponds to the baseboard you have. The file below corresponds to the Enclustra baseboard, which is the default and therefore is not specified in the title of the file. The file for the Nexsys baseboard is listed next. The below is an example only - please contact the Timing development team via the #timing-integration Slack channel on the DUNE workspace to check which version of the firmware to use. If you wish to use the endpoint as the Master to issue commands, the master firmware bitfile is the one to use. 

 ```
 wget https://pdts-fw.web.cern.ch/pdts-fw/ci/tags/v7.0.0/pipeline4610440/package/master_pc053d_v7-0-0_sha-3c684fa7_runner-slu9p8x4-project-19909-concurrent-0_221012_1128.tgz
 
 ```


 ```
 wget https://pdts-fw.web.cern.ch/pdts-fw/ci/tags/v6.2.0/pipeline3416367/ouroboros_pc053d_nexys_video_v6-2-0_sha-91dc3cf5_runner-slu9p8x4-project-19909-concurrent-13_220106_1707.tgz
 
 ```

Untar the directory and there will be a list of files including the relevant bit files.
Using Vivado or an alternative method, connect to the JTAG chain and program the FPGA with this firmware bit file. 


!!! warning
	Unless a specific IP address has been programmed on the FMC, RARP will be used to assign an IP address, i.e. the firwmare will use the Ethers file on the PC to assign IP addresses. You must edit this file to assign the MAC address of the baseboard (coming from the baseboard UID) to the IP you wish to assign it to. If you do not have RARP installed then you have to specify the IP via the COM port that the board is plugged into.

##Setup the connections file

A local connections file is the simplest method to ensure you connect to the correct board. A template of this file is listed here, called connections.xml

  ```

	<?xml version="1.0" encoding="UTF-8"?>

	<connections>
  	<connection id="PRIMARY"             uri="ipbusudp-2.0://192.168.200.16:50001" address_table="file://${TIMING_SHARE}/config/etc/addrtab/v7xx/master_fmc/top.xml" />
	</connections>

 
 ```

You can use the above template, but you will need to edit the file to point to the IP address assigned to the board. In addition the address table location has to point to the correct address table for the firmware you are using. The address table can be supplied by the timing development team when they instruct which firmware to use. 


##Configuring the FMC for operation at 62.5MHz

 Reset the FMC with a 62.5 MHz clock file; this will output the clock. The PRIMARY in the command below has to be specified in your connections file, these DEVICE IDs have to match. 

 ```
 dtsbutler -c connections.xml io PRIMARY reset  

 ```	

??? info 
	``` python
	Created device PRIMARY
	Design 'master' on board 'pc069' on carrier 'nexus-video' with frequency 62.5 MHz
	Resetting PRIMARY
	2023-Jun-07 14:49:11,171 LOG [dunedaq::timing::FMCIONode::reset(...) at /users/wx21978/projects/timing/daq_4.0.0/sourcecode/timing/src/FMCIONode.cpp:85] PLL configuration file : /home/wx21978/projects/timing/temp/pc069_clock_file.txt
	2023-Jun-07 14:49:13,570 LOG [dunedaq::timing::FMCIONode::reset(...) at /users/wx21978/projects/timing/daq_4.0.0/sourcecode/timing/src/FMCIONode.cpp:118] Reset done
	----------------Hardware info----------------
	+--------------------------+----------------+
	|        Board type        |      pc069     |
	|      Board revision      |     pc069a     |
	|         Board UID        | 0x803428749c04 |
	|       Carrier type       |   nexus-video  |
	|        Design type       |     master     |
	| Firmware frequency [MHz] |    62.500000   |
	+--------------------------+----------------+

	------FMC IO state-----
	+-------------+-------+
	|   Register  | Value |
	+-------------+-------+
	|   cdr_lol   |  0x0  |
	|   cdr_los   |  0x0  |
	|   mmcm_ok   |  0x1  |
	| mmcm_sticky |  0x0  |
	|   sfp_flt   |  0x0  |
	|   sfp_los   |  0x1  |
	+-------------+-------+


	PLL Clock frequency measurement:
	PLL freq: 250.002006528
	CDR freq: 0
	SMPL freq: 0


	PLL configuration id   : 069a_mst
	-------PLL information------
	+-----------------+--------+
	|     Register    |  Value |
	+-----------------+--------+
	|   Device grade  |   0x0  |
	| Device revision |   0x0  |
	|   Part number   | 0x5395 |
	+-----------------+--------+

	----------PLL state----------
	+-------------------+-------+
	|      Register     | Value |
	+-------------------+-------+
	|      CAL_PLL      |  0x0  |
	|        HOLD       |  0x1  |
	|        LOL        |  0x1  |
	|        LOS        |  0x0  |
	|      LOSXAXB      |  0x0  |
	|    LOSXAXB_FLG    |  0x1  |
	|        OOF        |  0x0  |
	|    OOF (sticky)   |  0xf  |
	|   SMBUS_TIMEOUT   |  0x0  |
	| SMBUS_TIMEOUT_FLG |  0x0  |
	|      SYSINCAL     |  0x0  |
	|    SYSINCAL_FLG   |  0x1  |
	|      XAXB_ERR     |  0x0  |
	|    XAXB_ERR_FLG   |  0x1  |
	+-------------------+-------+

```

Set the time of the FMC master block:

```
dtsbutler -c connections.xml mst PRIMARY synctime

```
??? info 
	``` python
		Created device PRIMARY
	Design 'master' on board 'pc069' on carrier 'nexus-video' with frequency 62.5 MHz
	----------------Hardware info----------------
	+--------------------------+----------------+
	|        Board type        |      pc069     |
	|      Board revision      |     pc069a     |
	|         Board UID        | 0x803428749c04 |
	|       Carrier type       |   nexus-video  |
	|        Design type       |     master     |
	| Firmware frequency [MHz] |    62.500000   |
	+--------------------------+----------------+

	Master FW rev: v7.0.0, channels: 5
	2023-Jun-08 11:34:18,205 LOG [dunedaq::timing::MasterNode::sync_timestamp(...) at /users/wx21978/projects/timing/daq_4.0.0/sourcecode/timing/src/MasterNode.cpp:295] Reading old timestamp: 0x9c4b92cd, Thu, 01 Jan 1970 01:00:41 +0000
	2023-Jun-08 11:34:18,207 LOG [dunedaq::timing::MasterNode::sync_timestamp(...) at /users/wx21978/projects/timing/daq_4.0.0/sourcecode/timing/src/MasterNode.cpp:298] Setting new timestamp: 0x1766a8929416240, Thu, 08 Jun 2023 11:34:18 +0000
	2023-Jun-08 11:34:18,207 LOG [dunedaq::timing::MasterNode::sync_timestamp(...) at /users/wx21978/projects/timing/daq_4.0.0/sourcecode/timing/src/MasterNode.cpp:303] Reading new timestamp: 0x1766a8929416e22, Thu, 08 Jun 2023 11:34:18 +0000
	2023-Jun-08 11:34:18,207 LOG [dunedaq::timing::MasterNode::sync_timestamp(...) at /users/wx21978/projects/timing/daq_4.0.0/sourcecode/timing/src/MasterNode.cpp:306] Timestamp broadcast enabled
	```

	Check status of timing master - - the TimeSync cnts number in the final table below shows the number of pulses sent.

	```
	dtsbutler -c connections.xml mst PRIMARY status 

	```

	??? info 
		``` python
		Design 'master' on board 'pc069' on carrier 'nexus-video' with frequency 62.5 MHz
	----------------Hardware info----------------
	+--------------------------+----------------+
	|        Board type        |      pc069     |
	|      Board revision      |     pc069a     |
	|         Board UID        | 0x803428749c04 |
	|       Carrier type       |   nexus-video  |
	|        Design type       |     master     |
	| Firmware frequency [MHz] |    62.500000   |
	+--------------------------+----------------+

	Master FW rev: v7.0.0, channels: 5
	Timestamp: 0x1766a893ba5014a -> Thu, 08 Jun 2023 11:34:22 +0000

	Master global controls
	+------------+-------+
	|  Register  | Value |
	+------------+-------+
	|  clr_ctrs  |  0x0  |
	|   resync   |  0x0  |
	| resync_cdr |  0x0  |
	|    ts_en   |  0x1  |
	+------------+-------+
	--Master global state-
	+------------+-------+
	|  Register  | Value |
	+------------+-------+
	| cdr_locked |  0x0  |
	|  ctrs_rdy  |  0x1  |
	|   rx_rdy   |  0x0  |
	|   ts_err   |  0x0  |
	|   tx_err   |  0x0  |
	+------------+-------+

	--------------Cmd gen counters--------------
	+------+-----------------+-----------------+
	|      | Accept counters | Reject counters |
	+------+-----------------+-----------------+
	| Chan |  cnts  |   hex  |  cnts  |   hex  |
	+------+--------+--------+--------+--------+
	|  0x0 |    0   |   0x0  |    0   |   0x0  |
	|  0x1 |    0   |   0x0  |    0   |   0x0  |
	|  0x2 |    0   |   0x0  |    0   |   0x0  |
	|  0x3 |    0   |   0x0  |    0   |   0x0  |
	|  0x4 |    0   |   0x0  |    0   |   0x0  |
	+------+--------+--------+--------+--------+

	--Master cmd counters (>0)-
	+-----+-------------------+
	|     | Sent cmd counters |
	+-----+-------------------+
	| Cmd |   cnts  |   hex   |
	+-----+---------+---------+
	| 0x0 |    4    |   0x4   |
	+-----+---------+---------+

	-Master acmd buffer-
	+----------+-------+
	| Register | Value |
	+----------+-------+
	|   ready  |  0x1  |
	|  timeout |  0x0  |
	+----------+-------+
	```

##Configuring the endpoint

First reset the Endpoint:

```
dtsbutler -c connections.xml io $EPT reset

```

Enable the address you wish to, in this example we enable address 0x2. This will put the endpoint into status 0x6 (WAITING FOR DELAYS).
    
```
dtsbutler -c connections.xml ept ${EPT} enable -a 2

```

Apply the timing delays, the first digit is the address, the second and third are coarse and fine delay parameters.

```
dtsbutler -c connections.xml mst ${MST} align apply-delay 2 0 0

```

Check the status of the endpoint.

```
dtsbutler -c connections.xml ept ${EPT} status

```













