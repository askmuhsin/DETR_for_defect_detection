# 🖌 Panoptic Head
<img width="712" alt="image" src="https://user-images.githubusercontent.com/8600096/158238904-4863efb0-ae96-4805-9967-8ef5caaaf8fe.png">

## 1. Where does the encoded image come from ?
Lets say the size of our input image is `3 X H X W`, where 3 is the RGB channel, H the height, and W the width. 

The image is first passed through a CNN backbone (in this case we use a ResNet50), this will extract a compact feature representation from the image, which will be of size `H/32 X W/32 X C`. Here C is the output vector size, which in this case is 2048. 

The next stage is to pass the featrues of `H/32 X W/32 X C` to an Encoder. Firstly a 1X1 convolution reduces the channel dimension from `C` to a smaller dimension `d`, creating a new feature map `d X H/32 X W/32`. At this stage positional embeddings are also added, as the transformer architecture is permutation-invariant. 

<img width="324" alt="image" src="https://user-images.githubusercontent.com/8600096/158238403-99f04a65-c8c7-4155-bfb6-9d389d7ab206.png">


## 2. Where is the d X N box embedding coming from ?
Once the encoded image is available it is passed through the decoder. The decoder is trained first to detect objects (bbox and corresponding class). In the DETR architecture we can have a set number of objects. This is the number of inputs to the decoder, they are called the object queries. For each object query the decoder outputs if there is an object or not, and when there is an object the output will have the class and bbox. This is done parallely for all the object queries. 

<img width="282" alt="image" src="https://user-images.githubusercontent.com/8600096/158289934-3dfb36e0-6778-46c6-bd04-df0d3e9880c0.png">

In panoptic segmentation task the object embeddings that has a class is set aside. The dimension of these object embeddings is `d X N`. Here N is number of the objects. 

## 3. How are the Attention maps generated ?
The multihead attention layer is used to generate the attention scores for the encoded image corresponding to each object embeddings.  


## 4. Where is the Resnet features coming from, in the FPN-style CNN network ?
In the First step when the image is passed through the CNN backbone, some of the select intermediate layers of actiavtion of the models is stored, namely Res2, Res3, Res4, and Res5. 

Later in the process when the outputs (attention maps) of the multi-head attention layer for each of the object embeddings are available, it is then passed throught an FPN-style CNN to upsample the attention maps. This FPN network uses the intermediate activations that was stored from the backbone network. The resulting maps are high resolution and contains binary logits representing weather it belongs to a class or not. 

## 5. The overall steps in training a Panoptic instance segmenetation using DETR. 
- Train DETR to predict bounding boxes around things and stuff.
- Once the detection model is trained, the weights are freezed. and a mask head is trained. (for around 25 epochs).
- The outputs from the network are high resolution maps corresponding to each of the classes.
- a pixel wise argmax operation can be done on all of the masks to produce the final panoptic segementation. 
