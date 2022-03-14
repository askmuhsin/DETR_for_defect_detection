## 1. Where does the encoded image come from ?
Lets say the size of our input image is `3 X H X W`, where 3 is the RGB channel, H the height, and W the width. 

The image is first passed through a CNN backbone (in this case we use a ResNet50), this will extract a compact feature representation from the image, which will be of size `H/32 X W/32 X C`. Here C is the output vector size, which in this case is 2048. 

The next stage is to pass the featrues of `H/32 X W/32 X C` to an Encoder. Firstly a 1X1 convolution reduces the channel dimension from `C` to a smaller dimension `d`, creating a new feature map `d X H/32 X W/32`. At this stage positional embeddings are also added, as the transformer architecture is permutation-invariant. 

