To enable testing and commissioning of the timing system, a timing FPGA mezzanine card(FMC) was designed and fabricated.  The FMC has SFP, Ethernet,  and  U.FL  connectors,  and  can  act  as  either  as  a  timing  master  or endpoint, depending on the firmware uploaded onto the carrier board FPGA. The carrier board can be either an Enclustra PX3/AX3 or a Digilent Nexsys Video card. There are different firmware images produced depending on which baseboard you have. 

The following instructions will allow you to set up and control the timing FMC. 

## Pre-requisites:

### Hardware: 

* Timing FMC hosted on an Enclustra Mars PM3  base  board  with  an  Enclustra  Mars  AX3  FPGA  module or Digilent Nexsys Video card
* JTAG  cable
* Linux machine (for control and firmware uploads) running CentOS 7


### Software: 

The ProtoDUNE timing butler (pdtbutler) is a python-based command line tool which provides the functionality to control the timing FMC.
The pdtbutler along with the timing software backend is part of the DUNE DAQ installation. In terms of which DAQ release to use, the following
twiki page lists the most recent tags; in general the most recent tag here is the one to use. 

https://github.com/DUNE-DAQ/daq-release/tags

To install this release you can find the instructions listed here on the right hand side column for the particular release you are targetting. 
Follow the instructions for Step 1 and Step 3.

https://github.com/DUNE-DAQ/daqconf/wiki


!!! warning
	You will need access to CVMFS to follow the above instructions. 

### Firmware 

Download the firmware bit file for the FMC which will allow the FMC to behave as a master timing unit - remember to download the file which corresponds to the baseboard you have. The file below corresponds to the Enclustra baseboard, which is the default and therefore is not specified in the title of the file. The file for the Nexsys baseboard is listed next. If the clock frequency is not specified in the file as 50 MHz, then the file will be 62.5MHz. The below is an example only - please contact the Timing development team via the #timing-integration Slack channel on the DUNE workspace to check which version of the firmware to use. 

 ```
 wget https://pdts-fw.web.cern.ch/pdts-fw/ci/tags/v6.2.0/pipeline3416367/ouroboros_pc053d_fmc_v6-2-0_sha-91dc3cf5_runner-slu9p8x4-project-19909-concurrent-5_220106_1706.tgz
 
 ```


 ```
 wget https://pdts-fw.web.cern.ch/pdts-fw/ci/tags/v6.2.0/pipeline3416367/ouroboros_pc053d_nexys_video_v6-2-0_sha-91dc3cf5_runner-slu9p8x4-project-19909-concurrent-13_220106_1707.tgz
 
 ```

Untar the directory and there will be a list of files including the relevant bit files.
Using Vivado or an alternative method, connect to the JTAG chain and program the FPGA with this firmware bit file. 


!!! warning
	The firwmare is currently using the Ethers file on the PC to assign IP addresses. You must edit this file to assign the MAC address of the baseboard (coming from the baseboard UID) to the IP you wish to assign it to. The IP address for the standard operation of an FMC, acting as master and/or endpoint is 192.168.200.16.



##Configuring the FMC for operation at 62.5MHz

 Reset the FMC with a 62.5 MHz clock file; this will output the clock.

 ```
 pdtbutler io PRIMARY reset  

 ```	

??? info 
	``` python
	Created device PRIMARY
	Design 'ouroboros' on board 'fmc' on carrier 'enclustra-a35'
	Resetting PRIMARY
	2021-03-09 16:16:56.276 pdt NOTICE   | PLL configuration file : Si5394-RevA-94mst625-Registers.txt
	2021-03-09 16:16:56.280 pdt NOTICE   | Configuring PLL        : SI5394
	2021-03-09 16:16:59.105 pdt NOTICE   | PLL configuration id   : 94mst625
	-----------Hardware info----------
	+----------------+---------------+
	|   Board type   |      fmc      |
	| Board revision |    kFMCRev3   |
	|    Board UID   | 0x49162b67ce6 |
	|  Carrier type  | enclustra-a35 |
	|   Design type  |   ouroboros   |
	+----------------+---------------+

	------FMC IO state-----
	+-------------+-------+
	|   Register  | Value |
	+-------------+-------+
	|   cdr_lol   |  0x1  |
	|   cdr_los   |  0x1  |
	|   mmcm_ok   |  0x1  |
	| mmcm_sticky |  0x0  |
	|   sfp_flt   |  0x1  |
	|   sfp_los   |  0x1  |
	+-------------+-------+


	PLL Clock frequency measurement:
	PLL freq: 312.490080593
	CDR freq: 36.4953250256


	PLL configuration id   : 94mst625
	-------PLL information------
	+-----------------+--------+
	|     Register    |  Value |
	+-----------------+--------+
	|   Device grade  |   0x0  |
	| Device revision |   0x0  |
	|   Part number   | 0x5394 |
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
pdtbutler mst PRIMARY synctime

```
??? info 
	``` python
		Created device PRIMARY
	-----------Hardware info----------
	+----------------+---------------+
	|   Board type   |      fmc      |
	| Board revision |    kFMCRev3   |
	|    Board UID   | 0x49162b67ce6 |
	|  Carrier type  | enclustra-a35 |
	|   Design type  |   ouroboros   |
	+----------------+---------------+

	Master FW rev: 0x50100, partitions: 4, channels: 5
	2021-03-09 16:35:30.928 pdt NOTICE   | Old timestamp: 0x1387088a1, Thu, 01 Jan 1970 01:01:44 +0000
	2021-03-09 16:35:30.929 pdt NOTICE   | New timestamp: 0x11eefb0ec7ae321, Tue, 09 Mar 2021 16:35:30 +0000
	```


Configure timing partition 0 - this will start sending out time pulses once per second.

```
pdtbutler mst PRIMARY part 0 configure

```

??? info 
	``` python
		Created device PRIMARY
	-----------Hardware info----------
	+----------------+---------------+
	|   Board type   |      fmc      |
	| Board revision |    kFMCRev3   |
	|    Board UID   | 0x49162b67ce6 |
	|  Carrier type  | enclustra-a35 |
	|   Design type  |   ouroboros   |
	+----------------+---------------+

	Master FW rev: 0x50100, partitions: 4, channels: 5

	Configuring partition 0
	Trigger mask set to 0xf1
	  Fake mask 0x1
	  Phys mask 0xf
	Partition 0 enabled and configured
	```

Check status of timing partition 0 - - the TimeSync cnts number in the final table below shows the number of pulses sent.

```
pdtbutler mst PRIMARY part 0 status 

```

??? info 
	``` python
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


	=> Partition 0

	---------Controls--------
	+---------------+-------+
	|    Register   | Value |
	+---------------+-------+
	|     buf_en    |  0x0  |
	|   frag_mask   |  0x0  |
	|    part_en    |  0x1  |
	|  rate_ctrl_en |  0x1  |
	|    run_req    |  0x0  |
	| spill_gate_en |  0x1  |
	|  trig_ctr_rst |  0x0  |
	|    trig_en    |  0x0  |
	|   trig_mask   |  0xf1 |
	+---------------+-------+

	--------State-------
	+----------+-------+
	| Register | Value |
	+----------+-------+
	|  buf_err |  0x0  |
	| buf_warn |  0x0  |
	|  in_run  |  0x0  |
	| in_spill |  0x0  |
	|  part_up |  0x1  |
	|  run_int |  0x0  |
	+----------+-------+

	Event Counter: 0
	Buffer status: OK
	Buffer occupancy: 0

	+-------------+-----------------+-----------------+
	|             | Accept counters | Reject counters |
	+-------------+-----------------+-----------------+
	|     Cmd     |  cnts  |   hex  |  cnts  |   hex  |
	+-------------+--------+--------+--------+--------+
	|   TimeSync  |   53   |  0x35  |    0   |   0x0  |
	|     Echo    |    0   |   0x0  |    0   |   0x0  |
	|  SpillStart |    0   |   0x0  |    0   |   0x0  |
	|  SpillStop  |    0   |   0x0  |    0   |   0x0  |
	|   RunStart  |    0   |   0x0  |    0   |   0x0  |
	|   RunStop   |    0   |   0x0  |    0   |   0x0  |
	|   WibCalib  |    0   |   0x0  |    0   |   0x0  |
	|   SSPCalib  |    0   |   0x0  |    0   |   0x0  |
	|  FakeTrig0  |    0   |   0x0  |    0   |   0x0  |
	|  FakeTrig1  |    0   |   0x0  |    0   |   0x0  |
	|  FakeTrig2  |    0   |   0x0  |    0   |   0x0  |
	|  FakeTrig3  |    0   |   0x0  |    0   |   0x0  |
	|   BeamTrig  |    0   |   0x0  |    0   |   0x0  |
	|  NoBeamTrig |    0   |   0x0  |    0   |   0x0  |
	| ExtFakeTrig |    0   |   0x0  |    0   |   0x0  |
	+-------------+--------+--------+--------+--------+
	```

##Configuring the endpoint

First reset the Endpoint:

```
pdtbutler io $EPT reset

```

Enable the address you wish to, in this exame we enable address 0x2.

    
```
pdtbutler ept ${EPT} 0 enable -a 2

```

Check the status of the endpoint.

```
pdtbutler ept ${EPT} 0 status

```













