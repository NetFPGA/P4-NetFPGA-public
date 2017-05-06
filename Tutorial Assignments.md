Tutorial Assignments
====================

There are currently three tutorial assignments created to allow new users to become familiar with the workflow:

1. **Switch_Calc** - using the switch as a calculator and key-value store

2. **TCP_Monitor** - using the switch to monitor the number of bytes transferred in each TCP connection

3. **In-band Network Telemetry (INT)** - implement basic support for INT to facilitate network measurements

---

Switch_Calc
-----------

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
![Alt text](switch_calc_ops.pdf "Optional title")

What to do:
-----------

To complete this assignment you will need to do the following:

1. Modify `$SUME_FOLDER/tools/settings.sh` to ensure that the `P4_PROJECT_NAME` environment variable is set to `tcp_monitor`. Run `$ source settings.sh`
2. **Complete switch_calc.p4** - a skeleton P4 program has been provided for you in `$P4_PROJECT_DIR/src`. `TopParser` and `TopDeparser` are already complete. `TopPipe` (i.e. the match-action pipeline) has all of the necessary tables and actions defined. Your job is to fill in the control flow to implement the switch_calc program.
3. **Complete gen_testdata.py** - 

Hints:
------