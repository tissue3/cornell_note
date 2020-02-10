The Log of Progress

### Jan 21th

There are some interesting paper to begin with.

- Accelerators:
  <https://arxiv.org/pdf/1704.04760.pdf>
  <http://compas.cs.stonybrook.edu/~mferdman/downloads.php/MICRO16_Fused_Layer_CNN_Accelerators.pdf>
  <https://www.cc.gatech.edu/~hadi/doc/paper/2016-cogarch-dnn_weaver.pdf>
  <https://arxiv.org/pdf/1809.04070.pdf>
  <https://people.csail.mit.edu/emer/papers/2016.06.isca.eyeriss_architecture.pdf>
  <https://arxiv.org/pdf/1708.04485.pdf>
- Simulators:
  <https://github.com/ARM-software/SCALE-Sim>
  <https://arxiv.org/abs/1811.02883> <https://arxiv.org/pdf/1912.04481.pdf>

### Jan 29th

- It seems [*DNN Dataflow Choice Is Overrated*](<https://arxiv.org/pdf/1809.04070.pdf>) models the DNN accelerator well.

  - We can represent **dataflow** with their model: X, Y (input), Fx, Fy (weight), C (channel), K (output)

  | Dataflow | Standing for             |
  | -------- | ------------------------ |
  | X\|Y     | output stationary        |
  | Fx\|Fy   | Weight stationary        |
  | Fy \| Y  | Row stationary (Eyeriss) |
  | C\|K     | Weight stationary (TPU)  |
  | ...      | No local reuse           |
  - Important factors according to the paper: **register file size**, **on chip storage sizes**.
  - The paper, along with some other papers, only compares energy efficiency. How about throughput and latency? 
  - When simulating, shall we just use the actual data reported in papers as the chips are already fabricated (Eyeriss, TPU)? How about nm technology? (The paper uses 28nm, Eyeriss 65nm, TPU 28nm)

- Other factors:

  - **Fusion of layers**.
  - **Data reuse rate**. Together with dataflow, we need to be able to calculate how many times each feature map/weight is reused and then decide **bandwidth** between global buffer to PE.  e.g. FC does not need involve reusing weight.
  - **Networks** that determine how global buffer and PEs are connected and communicated. e.g. hierarchical mesh network (Eyeriss v2), reduction tree.
  - **Sparsity**. Weight/Activation density should decide how the data are represented. This is especially important for inference where activation plays an important rule.

### Feb 9th

- Existing designs:
  - Eyeriss (power efficiency)
    - Logical dataflow: row stationary
    - Global buffer: 108kb. 25 banks
    - PE: 12x14.  spad, mac, control
      - scratch pad: filter 224bx16b (SRAM), ifmap 12bx16b (RF), psum 24bx16b (RF)
      - MAC (16b adder, 16 to 32b multiplier.)
    - Network: mesh? X/Y-Bus and PE neighbour bus in the form of [data, col]<col/row>; 64b  
    - Data gating logic: extra 12b Zero Buffer
  - TPU
    - Systolic dataflow
    - Unified buffer: 96kx256x8b = 24mb
    - Matrix multiply unit: 256x256x8b
  - DNN dataflow choice is overrated
    - Logical dataflow: not important
    - Double global buffer: 256kb
    - PE
      - RF: 16b  first level, 128b second level
    - Network: reduction tree/systolic array. then ...?
  - SCNN (favors high sparsity)
    - Logic dataflow: input stationary (K/Kc->C->W->H->Kc->R->S)
    - PE: 8x8, each has 4x4 multipliers, 32 adders. 
      - Weight FIFO: 50 entries(500b); 
      - Input array/output ram: 10kb each; 
      - Accumulator buffer: 32 entries(6kb) 
      - Network: 16x32 crossbar
  - MAERI (PE utilization, low latency favors small filter)
    - Dataflow: configurable, weight stationary
    - Prefetch buffer: 80KB
    - PE(mul switches = local buffers = adder switch + 1= simple switch + 1): 168/374
    - Network: argumented reduction tree

- A big issue is I don't know how to describe the structure, how elements are connected with abstraction.
  - specify tree/network/systolic array?

