# Suitability of Forward-Forward and PEPITA Learning to MLCommons-Tiny benchmarks
This repository contains the spreadsheet("analysis.ods") of the quantitative analysis performed for the paper "Suitability of Forward-Forward and PEPITA Learning to MLCommons-Tiny benchmarks". 


## Brief description
The objective of the analysis is to evaluate the performances in terms of memory usage and complexity of a learning procedure X on a certain model Y tackling a task T on a dataset D. The learning procedures are Backpropagation (BP), Forward-Forward (FF), PEPITA (PEP), MEMPEPITA (MPE). The models evaluated were respectively named DS-CNN, MobileNet, ResNet and AutoEncoder (AE). The datasets used were Speech Commands (SC), Visual Wake Words (VWW), Cifar10 and ToyADMOS. The precise structure
and the source code of these models are publicly available in the [mlcommons/tiny repository](https://github.com/mlcommons/tiny). For more information, read the original paper.

## Methodology
### Assumptions
The following assumptions were made: 
- Weights, biases and activations are quantized to INT8. The softmax layer, present only in the CNN models, is quantized to FLOAT32. The MACCs of the softmax layer should, therefore, be FLOP32. However, as the computations for the softmax layer are negligible in the economy of considered neural networks, they were assumed INT8 for the sake of simplicity. 
- When calculating peak RAM it was considered that to calculate a new layer it is not possible to overwrite an existing layer buffer in memory, but it is necessary to allocate a new buffer (which contributes to total RAM).
- Batch normalization layers, present in the original models, were not considered.

### MACCs procedure

As first step, a closed formula to calculate the MACCs needed by a generic layer (fully-connected, convolutional, depthwise convolutional, etc) to perform a certain operation was expressed. The operations considered are forward pass, backward pass, weight update and other operations specific to Forward learning procedures, such as normalization and goodness for Forward-Forward and error projection for PEPITA. In such a way, given the characteristics of a layer (e.g. number of channels in input, size of kernel), it is possible to calculate the MACCs required by a specific operation. Therefore, for each layer its characteristics were written down and for each operation the corresponding formula was applied to obtain the required MACCs.

![spreadsheet](figures/spreadsheet_example.png)

The MACCs relative to a certain operation were summed across all layers, to obtain the MACCs required by the entire model to perform the operation. Then, the number of total MACCs for a learning procedure were calculated by summing the MACCs of the operations involved in such procedure. This was done both for training and inference.

To describe the characteristics of a layer the following notation has been used.
- C_in: n° of channels of the previous layer
- C_out: n° of channels of the current layer
- H_in: height of the feature map of the previous layer 
- W_in: width of the feature map of the previous layer
- H_out: height of the feature map of the current layer 
- W_out: width of the feature map of the current layer
- H_ker: height of the kernel
- W_ker: width of the kernel
- H_stide: stride value in vertical direction
- W_stride: stride value in horizontal direction
In Fully-Connected layers, H_in and H_out were used to indicate the number of activations of, respectively, the previous and the current layer. 

### RAM procedure
The method for calculating the peak activations memory usage during training depending on the learning procedure used is shown in the following table.
| Learning procedure   | Activations during Training                 |
| :------------------: | :------------------------------------------:|
| BP and PEPITA        | sum of the activations buffers of all layers |
| FF                   | maximum value of the sum of the activation buffers of two consecutive layers plus the input sample |
| MEMPEPITA            | maximum value of the sum of the activation buffers of two consecutive layers and of the largest activation buffer between these two layers |

To obtain the peak total RAM during training, it is sufficient to add to the peak activations memory the memory occupied by weights. During inference, RAM is composed by only activation buffers. The RAM at inference time is the same for BP, unsupervised FF, PEPITA and MEMPEPITA and consists of the maximum value of the sum of the activation buffers of two consecutive layers, while supervised FF adds to it the input activations.

### Training and Inference latencies on MCUs

To calculate the training and inference latencies on the STM32H735G-DK and NUCLEO-G474RE MCUs for one sample, the MACCs previously computed were multiplied by the cycles per MACC and divided by the frequency of the processor featured by the MCU. Considering the n° of samples in the dataset and the epochs originally used to train the model, training time was calculated and expressed in hours.  


### Validation

Next, the Tensorflow Lite version of each MLCommons/Tiny benchmark has been profiled with the latest version of the tool freely available named [X-CUBE-AI](https://www.st.com/en/embedded-software/x-cube-ai.html). It provided automated analysis of the models, for the forward pass, by measuring the number of MACCs and the footprint of the activation buffers for each layer, which were used to confirm the results of the theoretical analysis. It is possible to use X-CUBE-AI from the web through [STM32Cube.AI Developer Cloud](https://stm32ai-cs.st.com/home)


## Citation
If you find "Suitability of Forward-Forward and PEPITA Learning to MLCommons-Tiny benchmarks" helpful for your research, please consider citing the paper.

```
@INPROCEEDINGS{10189239,
  author={Pau, Danilo Pietro and Aymone, Fabrizio Maria},
  booktitle={2023 IEEE International Conference on Omni-layer Intelligent Systems (COINS)}, 
  title={Suitability of Forward-Forward and PEPITA Learning to MLCommons-Tiny benchmarks}, 
  year={2023},
  volume={},
  number={},
  pages={1-6},
  doi={10.1109/COINS57856.2023.10189239}}
```
