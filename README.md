# IEEE-802.11-Simulation-Using-NS-2
Tutorial hướng dẫn mô phỏng mạng không dây với công cụ ns-2


_[Source: https://www.evanjones.ca/ns2/](https://www.evanjones.ca/ns2/)_

## What can I do with ns2?
- Wired network simulations
- Wireless network simulations
- Includes common protocols (TCP, RealAudio, etc)
- Many low level details are configurable (queuing policy, MAC protocol, etc)


## High Level Structure
- Library of C objects with TCL interfaces
  - Nodes
  - Routing Protocols
  - Application Protocols
  - etc.
- Connect the objects together via TCL


## Workflow
1. (Optional) Create custom ns2 object
2. Create TCL scenario file
3. Run the scenario to produce a trace file
4. Process the trace file to get results


## Basic Wireless Scenario
- Chain of 5 nodes, spaced 200m apart
- Use DSDV routing
- FTP transfer from one end to the other
- Want to measure the throughput


## Diagram
![image](https://www.evanjones.ca/ns2/scenario.png)  


## Options
~~~tcl
# ===============
# Define options 
# ===============
set val(chan)         Channel/WirelessChannel  ;# channel type 
set val(prop)         Propagation/TwoRayGround ;# radio-propagation model 
set val(ant)          Antenna/OmniAntenna      ;# Antenna type 
set val(ll)           LL                       ;# Link layer type 
set val(ifq)          Queue/DropTail/PriQueue  ;# Interface queue type 
set val(ifqlen)       50                       ;# max packet in ifq 
set val(netif)        Phy/WirelessPhy          ;# network interface type 
set val(mac)          Mac/802_11               ;# MAC type 
set val(rp)           DSDV                     ;# ad-hoc routing protocol  
set val(nn)           5                        ;# number of mobilenodes
~~~


## Set Up Simulator
~~~tcl
# Create simulator
set ns_    [new Simulator]
 
# Set up trace file
$ns_ use-newtrace
set tracefd [open simple.tr w]
$ns_ trace-all $tracefd
 
# Create the "general operations director"
# Used internally by MAC layer: must create!
create-god $val(nn)
 
# Create and configure topography (used for mobile scenarios)
set topo [new Topography]
# 1000x1000m terrain
$topo load_flatgrid 1000 1000
~~~

## Configure Nodes
~~~tcl
 $ns_ node-config -adhocRouting $val(rp) \
        -llType $val(ll) \
        -macType $val(mac) \
        -ifqType $val(ifq) \
        -ifqLen $val(ifqlen) \
        -antType $val(ant) \
        -propType $val(prop) \
        -phyType $val(netif) \
        -channel [new $val(chan)] \
        -topoInstance $topo \
        -agentTrace ON \
        -routerTrace ON \
        -macTrace OFF \
        -movementTrace OFF
~~~
 
##  Create Nodes
~~~tcl
 for {set i 0} {$i < $val(nn) } {incr i} {
        set node_($i) [$ns_ node]       
        $node_($i) random-motion 0  ;# disable random motion
        $node_($i) set X_ 0.0
        $node_($i) set Y_ 0.0
        $node_($i) set Z_ 0.0
}
 
$node_(0) set X_ 0.0
$node_(1) set X_ 200.0
$node_(2) set X_ 400.0
$node_(3) set X_ 600.0
$node_(4) set X_ 800.0
~~~
 
##  Source and Destination
~~~tcl
 # 1500 - 20 byte IP header - 40 byte TCP header = 1440 bytes
Agent/TCP set packetSize_ 1440 ;# This size EXCLUDES the TCP header
 
set agent [new Agent/TCP]
set app [new Application/FTP]
set sink [new Agent/TCPSink]
 
$app attach-agent $agent
 
$ns_ attach-agent $node_(0) $agent
$ns_ attach-agent $node_(4) $sink
$ns_ connect $agent $sink
~~~
 
##  Node Structure
![image](https://www.evanjones.ca/ns2/structure.png)  
 
##  Run It
~~~tcl
# 120 seconds of running the simulation time
$ns_ at 0.0 "$app start"
$ns_ at 120.0 "$ns_ halt"
$ns_ run
 
$ns_ flush-trace
close $tracefd
~~~
 
##  Trace File
~~~sh
s -t 0.000000000 -Hs 0 -Hd -2 -Ni 0 -Nx 0.00 -Ny 0.00 -Nz 0.00 -Ne -1.000000 -Nl AGT ...
r -t 0.000000000 -Hs 0 -Hd -2 -Ni 0 -Nx 0.00 -Ny 0.00 -Nz 0.00 -Ne -1.000000 -Nl RTR ...
~~~
- Each line represents a message being transferred
- Common fields: 
  - Event type (s = send, r = received, d = drop) 
  - Time stamp 
  - Source and destination 
  - XYZ co-ordinates of the node 
  - Network layer (AGT = agent, RTR = router, ...) 
  - ... many many others

 
## Miscellaneous Tools
- Both command line tools and a Python API [details](https://www.evanjones.ca/software/ns2tools.html) 
- ns2stats.py: Aggregate statistics 
- throughputOverTime.py: Throughput per second 
- stupidplot.py: Plot data from Python

