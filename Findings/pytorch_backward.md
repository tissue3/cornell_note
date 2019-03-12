## Self-defined Backward Propagation

As long as one uses data type and operations in pytorch, the tool will be able to automatically do the backward propagation. All you need to do is to add the parameters you want to train.

```python
import torch
	class MyLayer(torch.nn.Module):
    	def __init__(self):
            mytensor = torch.tensor(3,3)
            torch.nn.init.uniform(mytensor,0,1)
            #add parameter with nn.Parameter()
            #by default, it set requires gradient to true
            self.mytensor = torch.nn.Parameter(mytensor)
     	def forward(self,input):
            #use method provided by pytorch here
            #it supports autograd, namely, auto-gradient computation
            return input*self.mytensor
    
```



In most cases, as long as all methods and data types are provided pytorch, *loss.backward()* can be used to calculate pytorch algorithm. However, there are some situations we find gradient vanishes, or not behaves as expected. For instance,  the default gradient of *torch.round()* gives 0. Pytorch provides such backward propagation method because quantization is mathematically inconsistent and cannot be defined in a proper way. Similarly, *torch.clamp()*, a method that put the an constraint on range of input, has the same problem.

In this case, we need to __override__ the original backward function.

```python
import pytorch
class RoundGradient(torch.autograd.Function):
    @staticmethod
    def forward(ctx, x):
        return x.round()
    @staticmethod
    def backward(ctx, g):
        return g 

class ClampGradient(torch.autograd.Function):
    @staticmethod
    def forward(ctx, x):
        ctx.save_for_backward(x)
        return x.clamp(min=0,max=1)
    @staticmethod
    def backward(ctx, g):
        x, = ctx.saved_tensors
        grad_input = g.clone()
        grad_input[x < 0] = 0
        grad_input[x>1] = 1
        return grad_input
```



We can easily check the gradient function by printing the results of backward propagation. Notice we can only print gradient of __Parameter__, not arbitrary torch variable. Pytorch is so designed to save memory.

```python
import torch
import gradient #suppose we wrote our own gradient function in gradient.py
input = torch.nn.Parameter(torch.randn(2,6), requires_grad = True)
x = gradient.RoundNoGradient.apply(input)
x = x.sum()
x.backward()
print(input.grad) # this should give 1

print(x.grad) #this should give None becaue x is not a parameter
```



There is another data type which seems to have the same power as *torch.nn.Parameter( )*: *torch.autograd.Variable( )*. I am not sure the difference between these two functions though. As a safe type, *torch.nn.Parameter( )* always works.

