# xnor enhanced neural nets // Hasso Plattner Institute

A fork of the deep learning framework [mxnet](http://mxnet.io) to study and implement quantization and binarization in neural networks.

Our current efforts are focused on binarizing the inputs and weights of convolutional layers, enabling the use of performant bit operations instead of expensive matrix multiplications as described in:

- [BMXNet: An Open-Source Binary Neural Network Implementation Based on MXNet](https://arxiv.org/abs/1705.09864)

# Setup

We use ``cmake`` to build the project. Make sure to install all the dependencies described [here](docs/get_started/setup.md#prerequisites). 

Adjust settings in cmake (build-type ``Release`` or ``Debug``, configure CUDA, OpenBLAS, OpenCV, OpenMP etc.)  

```shell
$ git clone --recursive https://github.com/hpi-xnor/mxnet.git # remember to include the --recursive
$ mkdir build && cd build
$ ccmake .. # or cmake, or GUI cmake
$ make -j `nproc`
```

This will generate the mxnet library. To be able to use it from python, be sure to add the location of the libray to your ``LD_LIBRARY_PATH`` as well as the mxnet python folder to your ``PYTHONPATH``:
```shell
$ export LD_LIBRARY_PATH=<mxnet-root>/build
$ export PYTHONPATH=<mxnet-root>/python
```
### Docker

There is a simple Dockerfile that you can use to ease the setup process. Once running, find mxnet at ``/mxnet`` and the build folder at ``/mxnet/release``. (Be *warned* though, CUDA will not work inside the container so training process can be quite tedious)

```shell
$ cd <mxnet-root>/smd_hpi/tools/docker
$ docker build -t mxnet
$ docker run -t -i mxnet
```

You probably also want to map a folder to share files (trained models) inside docker (``-v <absolute local path>:/shared``).

# Usage

Our main contribution are drop-in replacements for the Convolution, FullyConnected and Activation layers of mxnet called **QConvoluion**, **QFullyConnected** and **QActivation**.

These can be used when specifying a model. They extend the parameters of their corresponding original layer of mxnet with ``act_bit``.

## Quantization

Set the parameter ``act_bit`` to a value between 1 and 32 to quantize the weights and activation to that bit width.

The quantization on bit widths ranging from 2 to 31 bit is available mainly for scientific purpose. There is no speed or memory gain (rather the opposite since there are conversion steps) as the quantized values are still stored in full precision ``float`` variables.

### Binarization

To binarize the weights first set ``act_bit=1``. Then train your network (you can use CUDA). The resulting .params file will contain binary weights, but still store a single weight in one float. 

To convert your trained and saved network, call the model converter with your ``.params`` file: 
```shell
$ <mxnet-root>smd_hpi/tools/model_converter mnist-0001.params
```

This will generate a ``.params`` and ``.json`` file with prepended ``binarized_``. This model file will use only 1 bit of runtime memory and storage for every weight in the convolutional layers.

We have example python scripts to train and validate [resnet18](smd_hpi/examples/binary-imagenet1k) (cifar10, imagenet) and [lenet](smd_hpi/examples/binary_mnist) (mnist) neural networks with binarized layers.

There are example applications running on iOS and Android that can utilize binarized networks. Find them in the following repos:
- [Android image classification](https://github.com/hpi-xnor/android-image-classification)
- [iOS image classification](https://github.com/hpi-xnor/ios-image-classification)
- [iOS handwritten digit detection](https://github.com/hpi-xnor/ios-mnist)

Have a look at our [source, tools and examples](smd_hpi) to find out more.

### Citing BMXNet

Please cite BMXNet in your publications if it helps your research work:

```shell
@article{HPI_xnor,
  Author = {Haojin Yang, Martin Fritzsche, Christian Bartz, Christoph Meinel},
  journal = {ArXiv e-prints},
  archivePrefix = "arXiv",
  eprint = {1705.09864},
  Title = {BMXNet: An Open-Source Binary Neural Network Implementation Based on MXNet},
  Year = {2017}
}
```

### Reference

- [XNOR-Net: ImageNet Classification Using Binary Convolutional Neural Networks](https://arxiv.org/abs/1603.05279)
- [Binarized Neural Networks: Training Deep Neural Networks with Weights and Activations Constrained to +1 or -1](https://arxiv.org/abs/1602.02830)
- [DoReFa-Net: Training Low Bitwidth Convolutional Neural Networks with Low Bitwidth Gradients](https://arxiv.org/abs/1606.06160)

