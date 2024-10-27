

@azazelazaza to utilize your Nvidia GPU with YOLOv8, you'll first need to ensure that your PyTorch installation is compatible with your CUDA and CuDNN versions. These are necessary for exploiting the GPU acceleration capabilities.

### If PyTorch is properly set up with CUDA, it should automatically utilize your GPU when using the YOLOv8 model.

### To further ensure that a specific GPU is used during computations, you can indicate the GPU device number using the torch.cuda.set_device(device_number) function. 
### Here device_number is the index number of your GPU, starting from 0.

You'll need to call this function before initializing your YOLOv8 model. That way, the model will be loaded onto the device specified.

Please ensure to handle any issues pertaining to CUDA memory or compatibility issues, should any arise during computations on the GPU. The specific handling would depend on the particular error witnessed.


--------

https://docs.ultralytics.com/help/FAQ/

Ultralytics YOLO can be run on a variety of hardware configurations, including CPUs, GPUs, and even some edge devices. However, for optimal performance and faster training and inference, we recommend using a GPU with a minimum of 8GB of memory. NVIDIA GPUs with CUDA support are ideal for this purpose.


https://github.com/ultralytics/ultralytics/issues/3084

import torch
torch.cuda.set_device(0)




@shlyahin thank you for reaching out to us!

The device argument is not available in the constructor of the YOLOv8 class. The device parameter was introduced in the YOLOv5 implementation, but it is not supported in YOLOv8.

To run YOLOv8 on GPU, you need to ensure that your CUDA and CuDNN versions are compatible with your PyTorch installation, and PyTorch is properly configured to use CUDA. Additionally, you can set the GPU device using torch.cuda.set_device(0) before initializing the YOLOv8 model.

Please let me know if you have any further questions or concerns!


-----

@Loukious if you have confirmed that your M2 MPS GPU is not recognized and you're getting only CPU usage despite following the previous instructions, it's possible that YOLOv8 does not currently support the MPS backend which is designed for Apple silicon (M1, M2 chips).

For Nvidia GPUs, using CUDA, typically, when a model is loaded in PyTorch and CUDA is available and properly installed, running the model should automatically set it to use the CUDA device without an explicit to(device) call. However, it can be a good practice to confirm device placement by explicitly setting the device like so:

import torch
from ultralytics import YOLO

# Check for CUDA device and set it
device = 'cuda' if torch.cuda.is_available() else 'cpu'
print(f'Using device: {device}')

# Load model
model = YOLO('path/to/your/model.pt').to(device)

# Now run inference or training
If torch.cuda.is_available() returns True, then the .to(device) call should place the model on the CUDA device, making it ready for GPU-accelerated operations.

Please ensure to follow the compatible PyTorch and CUDA installation instructions for your specific GPU and system configuration. If you experience further issues, it might be helpful to include the exact error messages or symptoms you're encountering to assist in troubleshooting the problem.

As always, please refer to the official Ultralytics documentation and resources provided for the most up-to-date guidance and support.

-------


Hey @josmyk! It looks like PyTorch isn't detecting your GPU, which is why you're seeing the error. This usually happens when PyTorch isn't installed with CUDA support or if there's a mismatch between PyTorch and CUDA versions. 😅

First, let's check if PyTorch with CUDA support is installed correctly. Run this in your Python environment:

import torch
print(torch.__version__)
print(torch.cuda.is_available())
If torch.cuda.is_available() returns False, you'll need to reinstall PyTorch with the correct CUDA version. You can find the appropriate command on the PyTorch installation page by selecting your setup details.

For example, for CUDA 11.3:

pip install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu113
Make sure to match the CUDA version with what's supported by your GPU and installed on your system. After reinstalling, try running your training command again. 🚀

Let me know how it goes!