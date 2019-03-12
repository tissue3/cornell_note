## GOMP_4.0 not found

One day I was trying to downsample an image with torch.nn.functional.interpolate, pytorch reports me that 

```shell
[AttributeError: module ‘torch.nn.functional’ has no attribute 'interpolate']
```

This is strange because the repo says [interpolate](https://pytorch.org/docs/stable/nn.html?highlight=interpolate#torch.nn.functional.interpolate) has already been implemented.

In an [issue](https://github.com/1adrianb/face-alignment/issues/110) discussion I found this function is introduced in 0.4.1, and my pytorch version is 0.4.0. 



To get update to latest version, `conda update pytorch torchvision` is not enough though. It will only keep your pytorch version at 4.0.

One should adds config before updating the pytorch.

```shell
conda config --add channels soumith
```



Updating doesn't solve the problem. Not all libraries get updated. 

```shell
ImportError: /path/to/python/site-packages/torch/lib/libgomp.so.1: version `GOMP_4.0' not found (required by /path/to/python/site-packages/torch/lib/libTH.so.1)
```

Let's fix this.

```
rm path/to/python/site-packages/torch/lib/libgomp.so.1
ln -s /usr/lib/libgomp.so.1 path/to/python/site-packages/torch/lib/libgomp.so.1
```





