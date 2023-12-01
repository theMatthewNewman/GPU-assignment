# Managing GPU usage

## nvidia-smi
To see what GPUs are available you can run 
```bash
nvidia-smi
```
This will list out the devices that are available and how much memory they are using.
Watching continuously for updates is also possible.
```bash
watch -n .5 "nvidia-smi | tail -n $(($LINES - 2))"
```

## CUDA_DEVICE_ORDER

By default Cuda orders its devices by speed with the fastest device being at 0.
This can be confusing because it doesn't match the id's from nvidia-smi.


For simplicity we can order our devices by PCI_BUS_ID, this will cause the CUDA_DEVICE_ORDER to match nvidia-smi.
```bash
export CUDA_DEVICE_ORDER=PCI_BUS_ID
```
Now our device ids match up!

## CUDA_VISIBLE_DEVICES
Setting CUDA_VISIBLE_DEVICES won't tell a program to run on a specified GPU.
Instead it will only change what devices are visible to the program. 
If only one device is visible than it will run on that selected GPU, otherwise it will run on the first one on the list.
This can either be done for the entire session
```bash
export CUDA_VISIBLE_DEVICES=2
```
or just a single program.
```bash
CUDA_VISIBLE_DEVICES=2 <program-you-want-to-run>
```

Making multiple devices visible looks like this.
```bash
export CUDA_VISIBLE_DEVICES=1,4,6
```

## Specifying which device to use inside of python.
It is possible to set these environment variables at the start of our python program.
```python
import os
os.environ["CUDA_DEVICE_ORDER"] = "PCI_BUS_ID"
os.environ["CUDA_VISIBLE_DEVICES"] = "3"
```
It is important to do this before importing tensorflow or pytorch.


## Automation

It may be possible to automate this process to pick the best GPU for the job.
We can Query nvidia-smi choose the device with the lowest memory usage and run the job on it.
```bash
export CUDA_VISIBLE_DEVICES=$(nvidia-smi --query-gpu=memory.free,index --format=csv,nounits,noheader | sort -nr | head -1 | awk '{ print $NF }'
```


## Notes
Setting these environment variables may change which GPU is being run on in a pytorch program for example.

```bash
CUDA_VISIBLE_DEVICES=1 <python-program>
```

```python
import torch
a = torch.Tensor(1).to("cuda:1")
```

This will throw an error because only 1 cuda device is visible to the program at "cuda:0".