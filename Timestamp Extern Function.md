Timestamp Extern
================

**Description** - generate a timestamp when requested. The bit width of the timer (and hence how often it wraps) is indicated by the bit width of the result field.

**Instantiation**:

```
@Xilinx_MaxLatency(1)
@Xilinx_ControlWidth(0)
extern void <name>_timestamp<D>(in bit<1> valid, out D result);
```

* `valid` - 1 means timestamp requested, 0 means not requested

* `result` - the resulting timestamp