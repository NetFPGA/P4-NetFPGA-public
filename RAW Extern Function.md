RAW Extern
==========

**Description** - Atomically read, add to, or overwrite an entry of a register array

**Instantiation**:
```
#define REG_READ 8w0
#define REG_WRITE 8w1
#define REG_ADD  8w2
@Xilinx_MaxLatency(1)
@Xilinx_ControlWidth(width(T))
extern void <reg_name>_reg_raw<T, D>(in T index,
                                     in D newVal,
                                     in D incVal,
                                     in bit<8> opCode,
                                     out D result);
```
* `index` - the array index to access. Note that the bit width of the index field dictates the depth of the register. For example, an index field with a bit width of 4 results in a register array with 16 entries.

* `newVal` - the new value to write into the register at the specified index. This field is only used if opCode == REG_WRITE.

* `incVal` - the amount to add to the register at the specified index. This field is only used if opCode == REG_ADD

* `opCode` - either REG_READ, REG_WRITE, or REG_ADD

* `result` - the latest value of the register at the specified index (i.e. after modification)

* `@Xilinx_MaxLatency` - allows P4 programmer to specify the number of clock cycles to complete the extern operation. This particular extern is designed to complete in 1 clock cycle.

* `@Xilinx_ControlWidth` - allows P4 programmer to specify the width of the address space allocated to this register. This should usually always be equal to the width of the index field so that the control-plane can access all register entries.