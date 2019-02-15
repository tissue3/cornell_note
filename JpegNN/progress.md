This document is to record things progress

### Jan 28th

- In Jpeg algorithm, not only 3) quantization loses precision, 1)RGB to YCbCr, 2) fourier transform also cause the same problem. In the document I read, 1) and 2) are supposed to be lossless however.

- Finish the naive version of model

### Feb 4

- Visualize the model
- Debug the model
- ToTensor scale the input by 1/255. I am concerned about the weight of quantization matrix
- Quantization matrix does not change much, it may has already stay in the local minimal, let's give it some random weights
- Another question is, how can I tell the tool "better" matrix (that minimize devision results in many sense)

### Feb 14

- Gradient was abnormal: it seems round() and clamp() are killing my gradient, so I rewrite them. Besides, I tried to scale all operations back to the range of 0 and 1. This is legal because rgb2ycbcr, dct are linear operations. Quantization, as long as I put the quantization matrix into the range of 0 and 1. It gives the same results.
- Backward propagation is working normally, the evidence is all quantize terms vanishes to 1, which elliminates all differences between graph after and before Jpeg compression. This arouses another problem of regularization.

- I try to regularize quantize terms. However, the quantization matrix goes to another extreme situation: either 1 for small row and col indexes, or 255 for large row and col indexes. 

- Another point confusing me is when the regulation term has huge difference in magnitude, e.g. 1000 vs. 1, to normal loss function. It seems that the loss function is still converging. One possible explanation is addition basically make the gradient calculation independent to each other. Then other weight still changes to the proper places as expected. Maybe I should stop training of all parameters except for the jpeg layer.
