# FuTil

- FIFO architecture: <https://www.youtube.com/watch?v=Nr8q5VW-mXI>
  - ![fifo](/Users/Cindy/Nextcloud/Cornell/research/Note/SeaShell/figure/fifo.png)
  - FIFO can just keep running. 
  - Actually all components can just keep running.

- **Enable** sematics:
  - It is fine to remove **enable**, just like we don't need **disable**. Currently FSM automatically goes from *end* to *start* stage, it is not done by **disable**. The transition from *start* to *execution* is also not because of **enable**, but a *reset* signal to the main component. The circuit can just keeps running and that's it.
- Dynamic scheduling (DS) vs. static scheduling (SS):
  - It seems that dynamic scheduling gets chance to perform better in latency whenever there is a cross loop *dependency*, at the expense of high resource usage(i.e. no resources reusage). 
  - If we want to exploit the advantage of dynamic scheduling, we need to combine it with static circuits. That is, only DS the dependent parts and SS the rest.

- Before the fancy features, there are some basic things we can do:
  - Change the inner representation to graph (e.g. CDFG).
  - Add passes to reduce unnecessary FSMs. (e.g. a+b; b\*c => (a+b)\*c)
  - But then what make FuTil different from other HLS?







