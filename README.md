# GPUstats
A Python module for getting the GPU status from NVIDA GPUs using `nvidia-smi`.
`GPUstats` locates all GPUs on the computer, determines their availablity and returns a ordered list of available GPUs.
Availablity is based opun the current memory consumption and load of each GPU.
The module is written with GPU selection for Deep Learning in mind, but it is not task/library specific and it can be applied to any task, where it may be useful to identify available GPUs.

**Table of Contents**

1. [Requirements](#requirements)
1. [Installation](#installation)
1. [Usage](#usage)
1. [Examples](#examples)
  1. [Select first available GPU in Caffe](#select-first-available-gpu-in-caffe)
  1. [Occupy only 1 GPU in TensorFlow](#occupy-only-1-gpu-in-tensorflow)
1. [License](#license)

## Requirements
CUDA GPU with latest CUDA driver installed.
GPUstats uses the program `nvidia-smi` to get the GPU status of all available CUDA GPUs. `nvidia-smi` should be installed automatically, when you install your CUDA driver.

Python libraries:
* [numpy](http://www.numpy.org/)
* subprocess ([The Python Standard Library](https://docs.python.org/3/library/))
* random ([The Python Standard Library](https://docs.python.org/3/library/))

Tested on CUDA driver version 367.57 and 375.26 with Python 3.5.2 and 3.5.4.

## Installation
1. Download or clone repository to your computer
2. Add GPUstats folder to ~/.bashrc
  1. Open a new terminal (Press Ctrl+Alt+T)
  2. Open bashrc:

      ```
      gedit ~/.bashrc
      ```

  3. Added your GPUstats folder to the environment variable `PYTHONPATH` (replace `<path_to_gpustats>` with your folder path):

      ```
      export PYTHONPATH="$PYTHONPATH:<path_to_gpustats>"

      Example:
      export PYTHONPATH="$PYTHONPATH:/home/anderskm/github/gpustats"
      ```

  4. Save ~/.bashrc and close gedit
  5. Restart your terminal
1. Test the installation
  1. Open a terminal in a folder other than the GPUstats folder
  2. Start a python console by typing `python` in the terminal
  3. In the newly opened python console, type:

    ```python
    import GPUstats
    GPUstats.showUtilization()
    ```

  4. Your output should look something like following, depending on your number of GPUs and their current usage:

    ```
     ID  GPU  MEM
    --------------
      0    0%   0%
    ```

## Usage

```python
GPUs = GPUstats.getGPUs()
```

```python
deviceIDs = GPUstats.getAvailable(order = 'first', limit = 1, maxLoad = 0.5, maxMemory = 0.5)
```

```python
GPUavailability = GPUstats.getAvailability(GPUs, maxLoad = 0.5, maxMemory = 0.5)
```

```python
deviceID = GPUstats.getFirstAvailable(maxLoad=0.5, maxMemory=0.5)
```

```python
GPUstats.showUtilization()
```

See [demo_GPUstats.py](https://github.com/anderskm/gpustats/blob/master/demo_GPUstats.py) for examples and more details

## Examples


### Select first available GPU in Caffe
In the Deep Learning library [Caffe](http://caffe.berkeleyvision.org/), the user can switch between using the CPU or GPU through their Python interface.
This is done by calling the methods `caffe.set_mode_cpu()` and `caffe.set_mode_gpu()`, respectively.
Below is a minimum working example for selecting the first available GPU with GPUstats to run a Caffe network.

```python
# Import caffe and GPUstats
import caffe
import GPUstats

# Get the first available GPU
DEVICE_ID_LIST = GPUstats.getFirstAvailable()
DEVICE_ID = DEVICE_ID_LIST[0] # grab first element from list

# Select GPU mode
caffe.set_mode_gpu()
# Select GPU id
caffe.set_device(DEVICE_ID)

# Initialize your network here

```

**Note:** At the time of writing this example, the Caffe Python wrapper only supports 1 GPU, although the underlying code supports multiple GPUs.
Calling directly Caffe from the terminal allows for using multiple GPUs.

### Occupy only 1 GPU in TensorFlow
By default, [TensorFlow](https://www.tensorflow.org/)  will occupy all available GPUs when using a gpu as a device (e.g. `tf.device('\gpu:0')`).
By setting the environment variable `CUDA_VISIBLE_DEVICES`, the user can mask which GPUs should be visible to TensorFlow via CUDA (See [CUDA_VISIBLE_DEVICES - Masking GPUs](http://acceleware.com/blog/cudavisibledevices-masking-gpus)). Using GPUstats.py, the CUDA_VISIBLE_DEVICES can be set programmatically based on the available GPUs.
Below is a minimum working example of how to occupy only 1 GPU in TensorFlow using GPUstats.
To run the code, copy it into a new python file (e.g. `demo_tensorflow_gpustats.py`) and run it (e.g. enter `python demo_tensorflow_gpustats.py` in a terminal).

```python
# Import os to set the environment variable CUDA_VISIBLE_DEVICES
import os
import tensorflow as tf
import GPUstats

# Get the first available GPU
DEVICE_ID_LIST = GPUstats.getFirstAvailable()
DEVICE_ID = DEVICE_ID_LIST[0] # grab first element from list

# Set CUDA_VISIBLE_DEVICES to first available device id
os.environ["CUDA_VISIBLE_DEVICES"] = str(DEVICE_ID)

# Store and print the device name
device = '/gpu:' + str(DEVICE_ID)
print('Device: ' + device)

# Run a minimum working example on the selected GPU
# Start a session
with tf.Session() as sess:
    # Select the device
    with tf.device(device):
        # Declare two numbers and add them together in TensorFlow
        a = tf.constant(12)
        b = tf.constant(30)
        result = sess.run(a+b)
        print('a+b=' + str(result))

```

Your output should look something like the code block below. Notice how only one of the GPUs are found and created as a tensorflow device.

```
I tensorflow/stream_executor/dso_loader.cc:128] successfully opened CUDA library libcublas.so locally
I tensorflow/stream_executor/dso_loader.cc:128] successfully opened CUDA library libcudnn.so locally
I tensorflow/stream_executor/dso_loader.cc:128] successfully opened CUDA library libcufft.so locally
I tensorflow/stream_executor/dso_loader.cc:128] successfully opened CUDA library libcuda.so.1 locally
I tensorflow/stream_executor/dso_loader.cc:128] successfully opened CUDA library libcurand.so locally
Device: /gpu:0
I tensorflow/core/common_runtime/gpu/gpu_device.cc:885] Found device 0 with properties: 
name: TITAN X (Pascal)
major: 6 minor: 1 memoryClockRate (GHz) 1.531
pciBusID 0000:02:00.0
Total memory: 11.90GiB
Free memory: 11.76GiB
I tensorflow/core/common_runtime/gpu/gpu_device.cc:906] DMA: 0 
I tensorflow/core/common_runtime/gpu/gpu_device.cc:916] 0:   Y 
I tensorflow/core/common_runtime/gpu/gpu_device.cc:975] Creating TensorFlow device (/gpu:0) -> (device: 0, name: TITAN X (Pascal), pci bus id: 0000:02:00.0)
a+b=42

```
Comment the `os.environ["CUDA_VISIBLE_DEVICES"] = str(DEVICE_ID)` line and compare the output. you output should look something like code block below. Notice, how all 4 GPUs are being found and created as a tensorflow device.

```
I tensorflow/stream_executor/dso_loader.cc:128] successfully opened CUDA library libcublas.so locally
I tensorflow/stream_executor/dso_loader.cc:128] successfully opened CUDA library libcudnn.so locally
I tensorflow/stream_executor/dso_loader.cc:128] successfully opened CUDA library libcufft.so locally
I tensorflow/stream_executor/dso_loader.cc:128] successfully opened CUDA library libcuda.so.1 locally
I tensorflow/stream_executor/dso_loader.cc:128] successfully opened CUDA library libcurand.so locally
Device: /gpu:0
I tensorflow/core/common_runtime/gpu/gpu_device.cc:885] Found device 0 with properties: 
name: TITAN X (Pascal)
major: 6 minor: 1 memoryClockRate (GHz) 1.531
pciBusID 0000:02:00.0
Total memory: 11.90GiB
Free memory: 11.76GiB
W tensorflow/stream_executor/cuda/cuda_driver.cc:590] creating context when one is currently active; existing: 0x2c8e400
I tensorflow/core/common_runtime/gpu/gpu_device.cc:885] Found device 1 with properties: 
name: TITAN X (Pascal)
major: 6 minor: 1 memoryClockRate (GHz) 1.531
pciBusID 0000:03:00.0
Total memory: 11.90GiB
Free memory: 11.76GiB
W tensorflow/stream_executor/cuda/cuda_driver.cc:590] creating context when one is currently active; existing: 0x2c92040
I tensorflow/core/common_runtime/gpu/gpu_device.cc:885] Found device 2 with properties: 
name: TITAN X (Pascal)
major: 6 minor: 1 memoryClockRate (GHz) 1.531
pciBusID 0000:83:00.0
Total memory: 11.90GiB
Free memory: 11.76GiB
W tensorflow/stream_executor/cuda/cuda_driver.cc:590] creating context when one is currently active; existing: 0x2c95d90
I tensorflow/core/common_runtime/gpu/gpu_device.cc:885] Found device 3 with properties: 
name: TITAN X (Pascal)
major: 6 minor: 1 memoryClockRate (GHz) 1.531
pciBusID 0000:84:00.0
Total memory: 11.90GiB
Free memory: 11.76GiB
I tensorflow/core/common_runtime/gpu/gpu_device.cc:777] Peer access not supported between device ordinals 0 and 2
I tensorflow/core/common_runtime/gpu/gpu_device.cc:777] Peer access not supported between device ordinals 0 and 3
I tensorflow/core/common_runtime/gpu/gpu_device.cc:777] Peer access not supported between device ordinals 1 and 2
I tensorflow/core/common_runtime/gpu/gpu_device.cc:777] Peer access not supported between device ordinals 1 and 3
I tensorflow/core/common_runtime/gpu/gpu_device.cc:777] Peer access not supported between device ordinals 2 and 0
I tensorflow/core/common_runtime/gpu/gpu_device.cc:777] Peer access not supported between device ordinals 2 and 1
I tensorflow/core/common_runtime/gpu/gpu_device.cc:777] Peer access not supported between device ordinals 3 and 0
I tensorflow/core/common_runtime/gpu/gpu_device.cc:777] Peer access not supported between device ordinals 3 and 1
I tensorflow/core/common_runtime/gpu/gpu_device.cc:906] DMA: 0 1 2 3 
I tensorflow/core/common_runtime/gpu/gpu_device.cc:916] 0:   Y Y N N 
I tensorflow/core/common_runtime/gpu/gpu_device.cc:916] 1:   Y Y N N 
I tensorflow/core/common_runtime/gpu/gpu_device.cc:916] 2:   N N Y Y 
I tensorflow/core/common_runtime/gpu/gpu_device.cc:916] 3:   N N Y Y 
I tensorflow/core/common_runtime/gpu/gpu_device.cc:975] Creating TensorFlow device (/gpu:0) -> (device: 0, name: TITAN X (Pascal), pci bus id: 0000:02:00.0)
I tensorflow/core/common_runtime/gpu/gpu_device.cc:975] Creating TensorFlow device (/gpu:1) -> (device: 1, name: TITAN X (Pascal), pci bus id: 0000:03:00.0)
I tensorflow/core/common_runtime/gpu/gpu_device.cc:975] Creating TensorFlow device (/gpu:2) -> (device: 2, name: TITAN X (Pascal), pci bus id: 0000:83:00.0)
I tensorflow/core/common_runtime/gpu/gpu_device.cc:975] Creating TensorFlow device (/gpu:3) -> (device: 3, name: TITAN X (Pascal), pci bus id: 0000:84:00.0)
a+b=42
```

## License
See [LICENSE](https://github.com/anderskm/gpustats/blob/master/LICENSE)
