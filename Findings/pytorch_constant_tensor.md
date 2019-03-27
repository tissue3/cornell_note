## How to support parallel computing with constant tensor in a self-defined layer?

In a self-defined layer, sometimes we want like to keep some constant local tensor. An intuitive way to do this is too add them as cuda tensor.

```python
class MyLayer(torch.nn.Module):
	def __init__(self):
		self.rgb2ycbcr = torch.cuda.FloatTensor([[.299,.587,.114],
                  [-0.168735892 ,- 0.331264108, 0.5],
                  [.5,- 0.418687589, - 0.081312411]])
```

However, this makes *self.rgb2ycbcr* automatically allocated to gpu 0. Since we would like to update each layer with this tensor, the program stucked.



Now take a look at [official tutorial](https://pytorch.org/tutorials/beginner/former_torchies/parallelism_tutorial.html). We find the layered get replication. 

```python
def data_parallel(module, input, device_ids, output_device=None):
    if not device_ids:
        return module(input)

    if output_device is None:
        output_device = device_ids[0]

    replicas = nn.parallel.replicate(module, device_ids)
    inputs = nn.parallel.scatter(input, device_ids)
    replicas = replicas[:len(inputs)]
    outputs = nn.parallel.parallel_apply(replicas, inputs)
    return nn.parallel.gather(outputs, output_device)
```



The [behavior](https://github.com/pytorch/pytorch/blob/master/torch/nn/parallel/replicate.py) of *nn.parallel.replicate(network, devices, detach=False)* is to replicate each parameter inside the network. Therefore we can add the local constant tensor to local parameter with requires_nograd set to False.

```python
class MyLayer(torch.nn.Module):
	def __init__(self):
		self.rgb2ycbcr = torch.nn.Parameter(torch.FloatTensor([[.299,.587,.114],
                  [-0.168735892 ,- 0.331264108, 0.5],
                  [.5,- 0.418687589, - 0.081312411]]))
```

Finally, when creating optimizer, we can filter the parameter to those only requires gradient calculation.

```python
param_require_grad = filter(lambda p: p.requires_grad, module.parameters())
```



 







