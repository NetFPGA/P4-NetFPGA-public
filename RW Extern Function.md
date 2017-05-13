RW Extern
=========

**Description** - Atomically read or overwrite an entry of a register array

**Instantiation**:
```
#define REG_READ 8w0
#define REG_WRITE 8w1
@Xilinx_MaxLatency(1)
@Xilinx_ControlWidth(width(T))
extern void <reg_name>_reg_rw<T, D>(in T index,
                                    in D newVal,
                                    in bit<8> opCode,
                                    out D result);
```
* `index` - the array index to access. Note that the bit width of the index field dictates the depth of the register. For example, an index field with a bit width of 4 results in a register array with 16 entries.

* `newVal` - the new value to write into the register at the specified index. This field is only used if opCode == REG_WRITE. 

* `opCode` - either REG_READ or REG_WRITE.

* `result` - the output, if REG_READ is specified then result will be equal to the current value of register at the specified index. If REG_WRITE is specified then result will be equal to newValue.

* `@Xilinx_MaxLatency` - allows P4 programmer to specify the number of clock cycles to complete the extern operation. This particular extern is designed to complete in 1 clock cycle.

* `@Xilinx_ControlWidth` - allows P4 programmer to specify the width of the address space allocated to this register. This should usually always be equal to the width of the index field so that the control-plane can access all register entries.