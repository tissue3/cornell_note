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
  - EIE (favors sparsity)
    - Logic dataflow:
    - PE: 256.
      - leadeing non-zero detection node(dynamically find non-zero elements)
      - activation queue
    - pointer read unit
      - ...
  - MAERI (PE utilization, low latency favors small filter)
    - Dataflow: configurable, weight stationary
    - Prefetch buffer: 80KB
    - PE(mul switches = local buffers = adder switch + 1= simple switch + 1): 168/374
    - Network: argumented reduction tree
  
- A big issue is I don't know how to describe the structure, how components are connected with abstraction.
  - specify tree/network/systolic array?
  - Or change the high level idea to: let user choose the design by providing the profiling results of utilization/latency/power consumption.
  - Or we can do a PE selection framework. Then the user can configure the quantity, network, size of RF, global buffer. 

### Mar 10th

- Recall we have 7 parameters: X, Y (output row/col), Fx, Fy (weight), C (channel), K (output), N (batch)
  - PE array should be 2D: we can map arbitrary parameters on each dimension. They are quick generator for mapping scheme, which will be explained later.
    - Eyeriss is Fy | Y (row stationary)
    - ShiDianNao is X | Y (output stationary)
    - SCNN is K,Fx,Fy | X,Y (input stationary)
  - In each PE, we need to describe what it does. I am not sure how we will take use of this. Currently it will only invoke some predefined visual modules. I also want it can perform simulation, but simulation requires dedicate design of input and output. Current index representation does not work.
    - For Eyeriss, output[0:X] = SeqAccumulate( SeqConv1D(w[0:Fx], input[0:X+Fx-1]) )
    - For ShiDianNao,  output = SeqAccumulate( SeqConv1D(w[0:Fx\*Fy], input[i:i+Fx\*Fy]) )
    - For SCNN, output\[k\]\[x\]\[y\] = TransformIndex( ParAccumulate(ParConv1D(w[0:F], input[0:I]))
  - For each PE, we need number of elements in each dimension are processed at 1) a single time step, 2) logically computation, i.e.  time-multiplexed so that no mapping will performed at PE array level.
    - For Eyeriss, at each time step, Fx inputs and Fx weight are stored. Logically, X of inputs and Fx of weights are processed. So when we map PE array, we don't consider X and Fx
    - For ShiDianNao, at each time step, 1 input and 1 weight is stored. Logically, Fx*Fy of both inputs and weights are processed.
    - For SCNN, at each time step, F inputs and I weights are stored. Logical computation is the same.
  - When performing 2D PE array mapping, parameters can be flattened by *f*. Flatten is opposed to the assumption that all parameters calculated sequentially. Parameters do not need to be fully flattened, or be multiple of the flattening factor.
    - Eyeriss can be flattened in Y, Fy, C, K, N. By default, Y and Fy are maximally flattened since the 2D mapping is set to Y and Fy. That is, for Y, *t* = if Y > PEArrayHeight then min(Y, int(PEArrayWidth/Fy)*PEArrayHeight)/1 else Y/1. However, it is fine to flatten it less, say PEArrayHeight/2. This leaves space for flattening multiple channels/output/batch of inputs to the PE array or replications of inputs so that different weights can be mapped at the same time.
    - For SCNN, on one dimension, *t* = Kc\*Fx\*Fy/F, while on the other, t = Xt*Yt/I  
  - All parameters can be replicated/reused *r* times across PE array. Again, the default version will maximally replicate parameters.
    - Eyeriss can be replicated in Y, Fy, C, K, N. By default, N, C and K are replicated by 1, i.e. not replicated. Fy (=3) is replicated Floor(13/3)=4 times by default, unless Y is too large to be squeezed in one pass. We can also replicated fewer times and flatten different channels, outputs and batches on it.
  - Replication, flattening and folding factors can be verified with each other and PE height/width.
  - Inside each PE, we can again set the three kinds of factors and verify them with local storage size. Notice flattening multiple channels does not result in increased psum storage. We can display it at some point.

- To sum up, we can describe PE array mapping with the following pseudo code. 

  - ```c++
    //Repeat this for each convolution layer
    Accel = Accelerator( X, Y, Fx, Fy, C, K, N)
    //load in hardware configure data
    Accel.LoadAccConfig(path_to_config_file) 
    
    //The auto mapper of Accel determines the default folding/ replication/ flattening factors for X, Y, Fx, Fy, C, K, N
    Param = Accel.getParameter()
    Param.add_param({"X","Y"}:"F", 5)//take F out of X*Y. X and Y are both not fully flattened in PE level, so we can perform more flattening in PE Array level
    PEMapping = PE(Step:(Param["Fx"], Param["Fx"]), Logic:("Fx","F"), tight=false, implementation = some function pointer) //tight=false allow perform multiplexing inside PE
    Accel.SetPEArrayMapping(H:"Fy", W:"Y", PE: PEMapping)
        
    //User can also set the factor by them self, but Accel has embeded checker to determine whether the factors are valid or not, i.e. following the hardware configuration and folding/ replication/ flattening rules.
    Accel.Height.SetFactors("X":{"t":2, "r":2}, "C":{"f":3})
    Accel.PE.SetFactors("Y":{"t":5,"r":2})
    Accel.verify()
    
    //Show the unrolling/tiling/flatten strategy with nested loops.     
    Accel.abstraction() 
    //one can image eventually something like this is displayed
    class PE(DefaultPE){
        InPELogic(weight, input){
            output[1:X-Fx+1] = accumulate( SeqConv1D(w([1:Fx], input[1:X]) )
            //we an also do ParConv1D, which creates multiple PEs.
        }
    }
    //generate some mapping strategy as shown in Eyeriss paper.
    Accel.visualize("out.png") 
    ```


- For configuration, we also care **global buffer**. This can be used to calculate power consumption for activation, weights, psum accesses. More importantly, how many layers can be **fused**. 
- Network is also important to send and receive data. A good network can send multiple data in one step. 
  - For example, Eyeriss v1 can only send the same data at each cycle and it takes multiple cycles to send out all weight/activations for the PE array. (Bus)
  - Eyeriss v2 came up with a way that every PE can receive data at the same time as long as they are in the same PE group (all-to-all). Data for other PE group can be transmitted with mesh network.
  - We can provide three network patterns: **bus, all-to-all, and mesh**. We can also group PEs and routers. For each wire connecting PE and routers we need to specify bandwidth to calculate latency. 

- TODO: Sparsity.

