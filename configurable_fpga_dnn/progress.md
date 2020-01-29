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
  - **Networks** that determine how global buffer and PEs are connected and communicated. e.g. hierarchical mesh network (Eyeriss v2).
  - **Sparsity**. Weight/Activation density should decide how the data are represented. This is especially important for inference where activation plays an important rule.