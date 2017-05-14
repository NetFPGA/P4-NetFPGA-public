[Home](https://bitbucket.org/sibanez/netfpga-sume-sdnet/wiki/Home)
------

---

P4 Developer Day Tutorial Assignments
=====================================

In this Lab session you will have the opportunity to complete the following three projects:

1. **Switch Calculator** - using the switch as a calculator and key-value store

2. **TCP Monitor** - using the switch to monitor the number of bytes transferred in each TCP connection

3. **In-band Network Telemetry (INT)** - implement basic support for INT to facilitate network measurements

---

Development Process
-------------------

To complete these assignments you will need to flush out the remainder of the provided P4 programs, run simulations to verify correct functionality, and perform hardware testing with a NetFPGA SUME board. You will do your development, debugging, and bitstream generation on a google compute instance. Then when you are ready to perform hardware testing, notify an instructor who will then give you access to a machine with a SUME board.

**Connecting to Google Compute Instance:**

* You should have received an email from an instructor that contains a private RSA key. You will use this key and an ssh client to connect to the Google compute instance. If you do not have the key, the instructors should have some USB sticks with the key. On UNIX workstations use the following command to connect to an instance:

    `$ ssh -i ~/.ssh/p4-netfpga-ssh-key p4user@<IP_ADDRESS>`

* Alternatively, if you prefer to work using a remote graphical desktop you can connect to the instance using a VNC client. Use the following commands to create a secure tunnel to the machine:

    `ssh -L 5901:localhost:5901 -i ~/.ssh/p4-netfpga-ssh-key p4user@<IP_ADDRESS>`

    Then open your VNC client and connect to `localhost:1` using the password provided by the instructors.

---

Assignment 1: Switch Calculator
-------------------------------

This is a simple tutorial which demonstrates many of the basic features of the P4-NetFPGA workflow. In this assignment, you will write a P4 program that configures the NetFPGA SUME switch to act as a simple calculator and key-value store. 

The operations that will be supported are:

* ADD - add two operands and return the result
* SUBTRACT - subtract two operands and return the result
* ADD_REG - add an operand to the current value stored in a register on the switch and return the result
* SET_REG - set the value of a register on the switch
* LOOKUP - lookup the given key in a table on the switch and return the result

In order to ask the switch to perform one of these operations, a client will send a packet with the following header on top of the Ethernet layer:
```
header Calc_h {
    bit<32> op1;
    bit<8> opCode;
    bit<32> op2;
    bit<32> result;
}
```
Where `op1` and `op2` are the first and second operand respectively, `opCode` indicates which of the 5 supported operations to perform, and the `result` field is set by the switch after performing the required computation. The switch should also swap the source and destination MAC addresses on the received packet before sending the final packet back to the client.

In summary, the switch will perform the following tasks:

* Receive and parse packet from client
* Swap the source and destination MAC addresses
* Examine the `opCode` field to determine the appropriate operation to perform
* Set the `result` field if necessary
* Construct the final packet and send it back to the client

The following image shows how the operands and registers should be used to perform the various functions:

![switch_calc_ops.png](https://bitbucket.org/repo/jp6axo/images/3452958472-switch_calc_ops.png)

What to do:
-----------

To complete this assignment you will need to do the following:

1. Modify `$SUME_FOLDER/tools/settings.sh` to ensure that the `P4_PROJECT_NAME` environment variable is set to `switch_calc`. Run `$ source settings.sh`

2. **Complete switch_calc.p4** - a skeleton P4 program has been provided for you in `$P4_PROJECT_DIR/src/switch_calc.p4`. `TopParser` and `TopDeparser` are already complete. `TopPipe` (i.e. the match-action pipeline) has all of the necessary tables and actions defined. Your job is to fill in the control flow to implement the switch_calc program. Note that the commands.txt file in the same directory fills the entries in the `lookup_table`.

3. **Review gen_testdata.py** - this is the python script that generates the test data (i.e. applied/expected packets and metadata) to be used in the simulations that verify functionality. 

4. Run the P4-SDNet compiler to generate the resulting HDL and an initial simulation framework: 

    `$ cd $P4_PROJECT_DIR && make`

5. Run the SDNet simulation: 

    `$ cd $P4_PROJECT_DIR/nf_sume_sdnet_ip/SimpleSumeSwitch`

    `$ ./vivado_sim.bash`. 

    Note: you may also run `vivado_sim_waveform.bash` if you would like to fire up the Vivado GUI and see the HDL waveforms (this is a very useful debugging tool).

    If this simulation passes great! If it does not you will need to modify either your P4 program or your `gen_testdata.py` script.

6. Generate the scripts that can be used in the NetFPGA SUME simulations to configure the table entries.

    `$ cd $P4_PROJECT_DIR && make config_writes`

7. Wrap SDNet output in wrapper module and install as a SUME library core:

    `$ cd $P4_PROJECT_DIR && make uninstall_sdnet && make install_sdnet`

8. Set up the SUME simulation. The `$NF_DESIGN_DIR/test/sim_switch_default` directory contains a `run.py` script which is responsible for running a SUME simulation, check it out. You will see that it reads the test packets generated by the `gen_testdata.py` script and applies the packets to SUME interfaces. All we need to do here is copy the `config_writes.py` script generated in step 6 into this directory.

    `$ cd $NF_DESIGN_DIR/test/sim_switch_default && make`

    Also run the `sim_switch_ctrlWrites` simulation, which demonstrates how to read/write registers in the SUME simulations. Remember to run `make` in this test directory as well.

9. Run the SUME simulation:

    `$ cd $SUME_FOLDER`

    `$ ./tools/scripts/nf_test.py sim --major switch --minor default`

    Note: you may also run the above command with the `--gui` option to fire up the Vivado GUI and see the HDL waveforms. Again, a very useful debugging tool.

10. Compile the bitstream:

    `$ cd $NF_DESIGN_DIR && make`

11. Copy the bitstream file and `config_writes.sh` script into the `$NF_DESIGN_DIR/bitfiles` directory.

    `$ cd $NF_DESIGN_DIR/bitfiles`

    `$ cp ../hw/project/simple_sume_switch.runs/impl_1/top.bit ./ && mv top.bit ${P4_PROJECT_NAME}.bit`

    `$ cp $P4_PROJECT_DIR/testdata/config_writes.sh ./`

12. Move to hardware test machine and program FPGA.

    On compute instance:

    `$ scp $NF_DESIGN_DIR/bitfiles/${P4_PROJECT_NAME}.bit p4user@<machine_name>.stanford.edu:$NF_DESIGN_DIR/bitfiles/`

    `$ scp $NF_DESIGN_DIR/bitfiles/config_writes.sh p4user@<machine_name>.stanford.edu:$NF_DESIGN_DIR/bitfiles/`

    `$ scp $P4_PROJECT_DIR/src/${P4_PROJECT_NAME}.p4 p4user@<machine_name>.stanford.edu:$P4_PROJECT_DIR/src/`

    On hardware testing machine:

    `$ cd $NF_DESIGN_DIR/bitfiles`

    `$ sudo bash`

    `# bash program_switch.sh`

    Note: Make sure that the configuration writes all succeed. If this is the first time programming the FPGA since the machine was last powered off it may need to be rebooted.

13. Test the design on real hardware! Go to the `$P4_PROJECT_DIR/sw/CLI` directory and run the `P4_SWITCH_CLI.py` script. This initiates an interactive command line interface that you can use to interact with your switch (i.e. read/write registers, add/remove table entries, etc.). Type `help` to see the list of available commands. 

    Go to the `$P4_PROJECT_DIR/sw/hw_test_tool` directory and run `$ sudo bash` then run the `switch_calc_tester.py` script. This will initiate a command line tool that you can use to submit packets to the switch and view its response. Type `help run_test` to see how to use the command.

    Try adding two numbers `testing> run_test 2 + 3` and seeing what you get. Also try doing things like setting one of the `const` register entries from the command line then submitting an `ADD_REG` packet. Or submit a `SET_REG` packet and read the value from the command line. Or add a new entry into the `lookup_table` and submit a `LOOKUP` packet to get the result.

14. If the switch appears to be working congratulations! You've finished the assignment!

Hints:
------

* Writing the P4 program:
    * You can check if your P4 program will compile by entering into the `$P4_PROJECT_DIR/src` directory and running `$ make`
    * You will need to use one of the register atom externs available in the P4-NetFPGA library. In this case, we have already instantiated the `const_reg_rw` atom for you, but you will need to use it in the control flow. These registers can only be accessed **one** time in the P4 code.

* Debugging tools:
    * While running the SDNet simulations, if you find that you have an error in the tuple (metadata) you can use the `$ $SUME_SDNET/bin/sss_sdnet_tuples.py --parse <binary_tuple_data>` command to see the sume_metadata values.
    * Use python scapy's `rdpcap()`, `Packet.show()`, and `hexdump()` commands to debug an error in the packet. Or alternatively, just go ahead and run the SUME simulation which will generate a file of logged and expected packets in `$NF_DESIGN_DIR/test`. Then use the `$ $SUME_SDNET/bin/nf_sim_compare_axi_logs.py --log <logged_file> --expect <expected_file>` to determine where the error is.

---

Assignment 2: TCP Monitor
-------------------------

This tutorial is designed to give users experience writing stateful P4 programs by utilizing some of the register atoms available in the P4-NetFPGA extern library.

In this assignment, you will write a P4 program to configure the NetFPGA SUME to perform some basic TCP connection monitoring. Recall that a TCP connection is established with the 3-way handshake (SYN, SYN-ACK, ACK) and in general, completes after a FIN packet is sent in both directions. For the purposes of this assignment, we will define a *flow* as all of the packets with a particular 5-tuple ID (srcIP, dstIP, protocol, source_port, destination_port); a flow starts with a packet that has the SYN bit set and ends with a packet that has the FIN bit set. Note that using this definition, each TCP connection consists of two flows, one in each direction.

The TCP Monitor P4 program will compute the size of each flow (in bytes) and after the flow has completed, it will update a histogram indicating the distribution of flow sizes that have passed through the switch. The control-plane can read this histogram off of the switch and display the current distribution in real time. 

The general idea is as follows:

* There are two register arrays:
    * `byte_cnt` - stores the current size in bytes of all "active" flows. Where an "active" flow is one in which the SYN packet has been seen, but the FIN packet has not been seen yet.
    * `dist` - stores the histogram of flow sizes that have been seen to pass through the switch.

* The 5-tuple is extracted from each arriving packet and hashed using a simple [longitudinal redundancy check](https://en.wikipedia.org/wiki/Longitudinal_redundancy_check) (LRC) to compute the index with which to access the `byte_cnt` register. 

* The initial SYN packet will reset the `byte_cnt` register entry to 0

* Each subsequent packet of the flow will increment its corresponding entry in the `byte_cnt` register with the size of its TCP payload.

* The FIN packet will extract the final size of the flow from the `byte_cnt` register and use it to increment one of the entries of the `dist` register.

* Note: It is true that the 5-tuple of different flows may hash to the same entry of the `byte_cnt` register, which will skew the resulting histogram towards larger flows. We will ignore that case for the purposes of this assignment to keep things simple.

What to do:
-----------

1. Modify `$SUME_FOLDER/tools/settings.sh` to ensure that the `P4_PROJECT_NAME` environment variable is set to `tcp_monitor`. Run `$ source settings.sh`

2. **Complete tcp_monitor.p4** - a skeleton P4 program has been provided for you in `$P4_PROJECT_DIR/src/tcp_monitor.p4`. `TopParser` and `TopDeparser` are already complete. `TopPipe` (i.e. the match-action pipeline) has all of the necessary tables and actions defined. Your job is to fill in the control flow to implement the tcp_monitor program. 

3. **Review gen_testdata.py** - this time we have completed the `gen_testdata.py` script for you, but ask that you review how it uses python's scapy module to generate the flows.

4. Follow steps 4 - 12 as listed in the "Switch Calculator" tutorial above.

5. Hardware testing! `$ cd $P4_PROJECT_DIR/sw/hw_test_tool` and open up two terminal windows:

    * In one terminal, run the `view_dist.py` script which will periodically read the flow size distribution from the switch and display it as a histogram.

    * In the other terminal, run `$ sudo bash` then run the `tcp_monitor_tester.py` script, which initiates a command line interface that allows you to send TCP flows through the switch

    Make sure that the histogram is being updated as you would expect. Note that if you try to send too many concurrent flows through the switch, some will hash to the same `byte_cnt` register entry which will skew the results.

6. If the switch appears to be working properly then congratulations! You've finished the assignment!

---

In-band Network Telemetry (INT)
-------------------------------

In this assignment you will write a P4 program to implement basic [INT](http://p4.org/p4/inband-network-telemetry/) support for the NetFPGA SUME platform. INT is quickly becoming one of the most popular applications for programmable data-planes, and it's all about gaining more visibility into your network. There are different ways of adding support for INT, but in this tutorial we will roughly base our implementation on the [current INT specification](http://p4.org/wp-content/uploads/fixed/INT/INT-current-spec.pdf). 

The basic idea is as follows:

* An INT source end host will generate a packet with an INT header (over Ethernet) that contains an instruction bitmask.

* Each bit of that instruction bitmask corresponds to a different type of metadata in the switch (e.g. switch ID, ingress port, egress port, ingress timestamp, etc.).

* If a particular bit is set in the instruction bitmask that means the switch should insert the corresponding metadata into the packet. Each piece of metadata is represented as a header with a single bottom-of-stack (`bos`) bit followed by 31 bits of data. The last piece of INT metadata must have the `bos` bit set to 1. The diagram below shows how new INT metadata is inserted into the packet.

```
|  Ethernet  |  INT  |  INT_data  |  INT_data  |  payload  |
|            |       |    bos:0   |    bos:1   |           |
                     ^ 
                     |
            New data inserted here
```

* The format of the INT header is shown below. We will ignore the replication and copy fields to simplify the implementation. 

```
// INT header
header int_h {
    bit<2> ver;                   // version #
    bit<2> rep;                   // replication requested
    bit<1> c;                     // is copy 
    bit<1> e;                     // max hop count exceeded
    bit<5> rsvd1;                 // reserved 1 
    bit<5> ins_cnt;               // # of 1's in instruction bitmask
    bit<8> max_hop_cnt;           // max # hops allowed to add metadata
    bit<8> total_hop_cnt;         // # hops that have added metadata 
    bit<5> instruction_bitmask;   // which metadata to add to packet
    bit<27> rsvd2;                // reserved 2
}
```

* Our INT implementation will support gathering 4 types of metadata from the switch:
    * SwitchID
    * Ingress Port
    * Egress Port
    * Ingress Timestamp

* Upon receiving the final packet, the INT sink end host will extract the resulting metadata.

What to do:
-----------

1. Modify `$SUME_FOLDER/tools/settings.sh` to ensure that the `P4_PROJECT_NAME` environment variable is set to `int`. Run `$ source settings.sh`

2. **Complete int.p4** - a skeleton P4 program has been provided for you in `$P4_PROJECT_DIR/src/int.p4`. 

3. **Review gen_testdata.py**

4. Follow steps 4 - 12 as listed in the "Switch Calculator" tutorial above.

5. Hardware testing! `$ cd $P4_PROJECT_DIR/sw/hw_test_tool` and open up two terminal windows:

    * In one terminal run `$ sudo bash` then run `# ./rcv_int`, which receives, parses, and prints received INT packets.

    * In the other terminal run `$ sudo bash` then run `# ./int_tester.py`, which allows you to send INT packets with a particular instruction bitmask into the switch. For example, `testing> run_test 0b11001` will submit an INT that requests to switch to insert the switchID, ingress port, and egress port into the packet. Type `testing> help run_test` to see documentation about the command.

If the switch appears to be working congratulations! You've completed the assignment.