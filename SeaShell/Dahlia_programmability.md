### Programmability of Dahlia
- There are cases that some loop is not unrollable but if we split part of it out, we have it.

  - This loop is not unrollable since it involves dynamic indexing of array `b`.

    ```Dahlia
    for (let r = 0..64){
    	a = b[c[r]];
    	m[r] = a + c[r];
    }
    ```

  - However, it would be fine if we split it into two loops, we might get some kind of speed up since the second part is unrollable now.

    ```Dahlia
    for (let r = 0..64){
    	a[r] = b[c[r]];
    }
    view a_v = a[_: bank 8]
    for (let r = 0..64) unroll 8{
    	m[r] = a[r] + c[r];
    }
    ```

- There are cases that the loop is not satisfying:

  - We can only unroll by 2 in this case:

    ```Dahlia
    for (let r = 0..62)unroll 4{//wrong! Dahlia does not allow that
     ...
    }
    ```

  - But we can also increase loop number. But there could be a tradeoff since we introduce more hardware and need to recover original values.

  ```Dahlia
  for (let r = 0..64) unroll 4{
   a[r] := b[r] + c[r];
  }
  for (let r = 63 ..64){
   ...
   //recover a[r] to its original value here
  }
  ```

  

- Trade of between inner loop and outer loop.

  - Examples can be found here - https://github.com/cucapra/fuse-benchmarks-sdaccel/blob/master/experiments/dahlia-dse/machsuite-stencil-stencil2d-inner/stencil.fuse and here -
    https://github.com/cucapra/fuse-benchmarks-sdaccel/tree/master/experiments/dahlia-dse/machsuite-stencil-stencil2d-outer/stencil.fuse

  - Notice HLS can do the second case automatically (completely unroll inner loops and then partially unroll outer loops) but in Dahlia we need to specify it.

    
