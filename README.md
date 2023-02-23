# On the Suitability of Forward-Forward learning to Tiny Devices
This repository contains the detailed descriptions of the quantitative analysis of 'Forward-only' algorithms on MLPerf Tiny benchmark.

## Procedure
We have first written closed formulas for the number of SUMs, MULs and MACCs for a generic layer (fully-connected, convolutional, depthwise convolutional, etc) during forward pass, backward pass and weight update. Other operations specific to the Forward learning procedure were considered, such as normalization, goodness and error projection. Then, we applied them to each layer and obtained the total MACCs for each learning procedure starting from the MACCs of the single operations.
To compute training and inference time on a certain MCU model, the corresponding MACCs were multiplied by the cycles per MACC and divided by the frequency of the processor.

## Assumptions
Several assumptions were made: 
- Weights, biases and activations are quantized to INT8. The softmax layer, present only in the CNN models, is quantized to FLOAT32. The MACCs of the softmax layer should, therefore, be FLOAT32. However, as the computations for the softmax layer are negligible in the economy of the NN, they were assumed INT8 for the sake of simplicity. 
- When calculating peak RAM we considered that to calculate a new layer you cannot overwrite an existing layer buffer in memory, but you needed to allocate a new buffer (which contributes to total RAM).


