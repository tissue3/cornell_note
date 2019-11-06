### Predictable Accelerator Design with Time-Sensitive Affine Types
- Why Fuse generates C/C++ codes since it's designed for Verilog?  
- If "traditional HLS tools...allow uneven banking and silently insert additional hardware to account for the complicated access patterns that it induces". Why this time we won't have any complicated access pattern?
- A[1] is equivalent to A{1}[0]. How about A[3] given decl A:float[10 bank 2]?
- A question I remember asked by Chris was even though Fuse can constrain the syntax to be "safe", in real world, it would still be contrained by FPGA resources. 
- Highligted blocks? suffix or prefix?

### Review: Predictable Accelerator Design with Time-Sensitive Affine Types
- Abstract
  - *Despite its restrictiveness, Dahlia admits hardware implementations that are nearly as efficient as designs from traditional HLS.* Sounds weak. It sounds like we are trying to compare HLS and Dahlia in efficiency, which is misleading.
  - _it can express 18 benchmarks from MachSuite._ I guess this will be modified anyway.
  - Try to conclude "even though Fuse can constrain the syntax to be "safe", in real world, it would still be contrained by FPGA resources, why do people want to use Dahlia?"
- Intro
  - _it is too explicit and verbose for productive engineering in most cases_ Yes, give example whenever trying to illustrate a point.
  - If possible, can we add graph of HLS because PL people may not know about it?
  - Talk more about HLS limitations. Examples/experiments like when it goes wrong. 
  - _Previous research has shown how to apply sub-structural type systems to model classic computational re-sources such as memory allocations and file handles [24, 9, 36, 49] and to enforce exclusion for safe shared-memory par-allelism [23, 8, 14]._ ???
  - _imperative erasure_ What does that mean?
- HLS vs Dahlia
  - Figure 2(a). in[10] -> out[10]
  - How about a type to compare cycles and hardware overhead for three designs?
  - What will happen if no array partition is placed?
  - In design principle (2.2), is there a way to mention `view` and order composition (`;` and `---` )as one of the important selling point? I feel why do we want to include have order composition and why do we want to use view should be reflected in design principle. What `time-sensitive affine types` is not mentioned here?
  - Why design principle belongs to section HLS vs Dahlia? Can we summarize with a table to learn the difference between HLS and dahlia?
- 

