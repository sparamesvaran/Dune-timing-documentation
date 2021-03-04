To enable testing and commissioning of the timing system, a timing FPGA mezzanine card(FMC) was designed and fabricated.  The FMC has SFP, Ethernet,  and  U.FL  connectors,  and  can  act  as  either  as  a  timing  master  or endpoint, depending on the firmware uploaded onto the carrier board FPGA.

The following instructions will allow you to set up and control the timing FMC. 

## Pre-requisites:

### Hardware: 

* Timing FMC hosted on an Enclustra Mars PM3  base  board  with  an  Enclustra  Mars  AX3  FPGA  module
* JTAG  cable
* Linux machine (for control and firmware uploads) running CentOS xxx


### Software: 

The ProtoDUNE timing butler (pdtbutler) is a python-based command line tool which provides the functionality to control the timing FMC.
The  pdt-butler has the following software dependencies:  boost libraries (v1.53), python 2.7, and the uhal package (installation instructions can be found from https://ipbus.web.cern.ch/ipbus/doc/user/html/software/installation.html). 




## Installation instructions for the ProtoDUNE timing software 

To set up the pdtbutler, please execute the following steps on the control machine.

* Clone the software repository, (ssh key access to gitlab required) 

	* `#!python git clone https://github.com/DUNE-DAQ/timing-board-software.git`

* Checkout FMC set-up tag

	* * `#!python git checkout relval/v6.0.0/b2`


* Compile the C++ layer
	*   `#!python make`

* Source the environment
	*  `#!python source tests/env.sh`


### Firmware 

The timing firmware repisitory is located here: https://pdts-fw.web.cern.ch/pdts-fw

Download the firmware bit file for the FMC which will allow the FMC to behave as a master timing unit

 ```
 wget xxx
 
 ```

Using Vivado or an alternative method, connect to the JTAG chain and program the FPGA with the firmware bit file. 

##Configuring the FMC

 Reset the FMC:

 ```
 pdtbutler io PRIMARY reset

 ```	
Set the time of the FMC master block:

 `#!python pdtbutler mst PRIMARY synctime`

 Configure timing partition 0

`#!python pdtbutler mst PRIMARY part 0 configure`

Check status of timing partition 0

`#!python pdtbutler mst PRIMARY part 0 status`

??? question "What should the status command return?"

    












