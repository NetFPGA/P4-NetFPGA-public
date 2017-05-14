PRAW Extern
===========

**Description** - Atomically add to or overwrite the register at the specified index if a predicate is true. Otherwise, read the current value.

**Instantiation**:
```
#define REG_READ 8w0
#define REG_WRITE 8w1
#define REG_ADD  8w2

#define EQ_RELOP    8w0
#define NEQ_RELOP   8w1
#define GT_RELOP    8w2
#define LT_RELOP    8w3
@Xilinx_MaxLatency(1)
@Xilinx_ControlWidth(width(T))
extern void <reg_name>_reg_praw<T, D>(in T index,
                                      in D newVal,
                                      in D incVal,
                                      in bit<8> opCode,
                                      in D compVal,
                                      in bit<8> relOp,
                                      out D result,
                                      out bit<1> boolean);
```

* `index` - the array index to access. Note that the bit width of the index field dictates the depth of the register. For example, an index field with a bit width of 4 results in a register array with 16 entries.

* `newVal` - the new value to write into the register at the specified index. This field is only used if opCode == REG_WRITE.

* `incVal` - the amount to add to the register at the speicifed index. This field is only used if opCode == REG_ADD
opCode - either REG_READ, REG_WRITE, or REG_ADD

* `compVal` - the value to compare to the current value of the register at the specified index

* `relOp` - the operation to use to compare compVal to the current value of the register at the specified index. compVal and relOp together define the predicate that decides whether or not the register value will be modified. If the predicate is true then the register may be modified, otherwise it will not be.

* `result` - the latest value of the register at the specified index (i.e. after modification)

* `boolean` - bit that indicates whether or not the predicate was true (1) or false (0)

* `@Xilinx_MaxLatency` - allows P4 programmer to specify the number of clock cycles to complete the extern operation. This particular extern is designed to complete in 1 clock cycle.

* `@Xilinx_ControlWidth` - allows P4 programmer to specify the width of the address space allocated to this register. This usually be equal to the width of the index field so that the control-plane can access all register entries.