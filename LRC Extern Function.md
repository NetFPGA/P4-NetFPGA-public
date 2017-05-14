LRC Extern
==========

**Description** - very simple hash function. Split the input data into multiple words and XOR each word together to obtain the result.

**Instantiation**:

```
@Xilinx_MaxLatency(1)
@Xilinx_ControlWidth(0)
extern void <name>_lrc<T, D>(in T in_data, out D result);
```

* `in_data` - the data to hash

* `result` - the resulting hash of in_data, which is obtained by splitting up in_data into words of size width(D) and XORing each word together.