# OPENPOSE WITH MOBILE_NET_V2 NETWORK

## Summarize
- Aim - Building support system for the elders.
- Method - Using `Openpose` for post estimation and lightweight netral network to detect falls.
> Use `Openpose` to extract human keypoints and label them in the images. After that, a `modified MobileNetV2 network` is used to detect falls by `integrating both keypoint information and pose information` in the original images. The above operation can use the original image to correct the deviation in the keyypoint labeling process.
- Results
    + Dataset UR - Accuracy: 98.6%
    + Dataset Le2i - Accuracy: 99.75%

## Current State Of The Field Of Study
### Kinds of Researches of FallDetection
- I. Using body aspect ratio, head position, acceleration, defections...
    + Taking change Human boday contour as Input features
    + `Head position`, `Body aspect ratio` Motion history Image `MHI` => Human posture information
    ```
    Fall detection based on RetinaNet
    and MobileNet convolutional neural networks. In: 2020 15th International
    Conference on Computer Engineering and Systems (ICCES), pp. 1–7.
    IEEE, Piscataway (2020)
    ```
    + [Body proportion, acceleration, deflection => Key features](https://ieeexplore.ieee.org/document/9638689)
    + [Fall motion vector modeling](https://ieeexplore.ieee.org/document/9437213)

- II. Extracting spatio-temporal features.
    + Using deep learning Algorithm to extract `spatiotemporal features` of the detected target
    + [Extract the Skeleton information of human body (Openpose) and the Falls were identified through three critical parameters](https://www.mdpi.com/2073-8994/12/5/744)
    + [Ultilize 3D CNN to extract spatio-temporal inforamtion from videos/images to detect stumbles](https://ieeexplore.ieee.org/document/8295206)

### Picking Stacks
```
Adopt Bottom-Up Algorithm Openpose and MobileNetV2 for this survey.
```
#### Why MobileNetV2 ?
Fast operation speed => more applicable => adopt the MoibleNetV2 CNN Architecture (fewer parameters for classification algorithm).

#### Why Bottom-Up Openpose ?
- `Top-Down` - Finding bounding boxes for each Person, Estimate human joint (keypoints) per bounding box
- `Bottom-Up` - Dectects key points firstly then associates the key points with a human body => Faster than the `Top-Down`

**Keypoints and Torso informaiton of the human body are marked based on retaining the original image information and the Feature enhancement operation is performed. Then, Deep learning algorithm of MobileNetV2 is used for feature extraction and finally detected falls.**

## Contributions.
- [1].  Keypoints are marked using openpose, `Marked images` are used as Input to classification network.
- [2]. Modified MobileNetV2 with a `fully connected layer` and a `softmax function` to preserve more information for detection task
- [3]. Improve accuracy of labeling human keypoints in dark environments, highlighted the column of verly dark frames in the `UR dataset`.

## Strategy
Features Enhancement and Fall detection.
### 1. Feature Enhancemen
`Problem`: original light in video data,there may be some bias in the key point extraction => Accuracy of the result may be affected if using only keypoint information.

`Solution`: Openpose is used to extract human keypoint information => Make key point **annotated**  in the original image. => Annotated Image used as input to MobileNetV2.

> The process involves preprocessing the images, extracting features using the first 10 layers of VGG-19, and predicting position confidence maps and part affinity fields for limbs. The network model parameters are optimized according to the loss function, associating body parts with the human body, and marking the keypoints in the original image. The method begins with torso identification and proceeds to connect body part points in the image. This enables the quick identification of body parts belonging to the same person, leading to the formation of a human frame. To distinguish it from the human body in the picture, colorful line segments and solid dots are used to mark the human torso. The study also analyzes the potential impact of marker color and size on detection results

<img src="./imgs/openpose_flowchart.png">

### 2. Fall Detection
> This section only describe the modifications to original framework and how they can be applied in the fall to detection scenario.

#### Experiments
- Using `Le2i dataset`

Architectures   | Accuracy  |
| -----------   | ----------|
MobileNetV2     | 98.5%
Efficientnet | 93.5%
EfficientnetV2 | 94.92%

- Using `UR dataset`

Architectures | Accuracy
|-|-|
MobileNetV2 | 96.3%
Efficientnet | 95.93%
EfficientnetV2 | 96%

#### Modification To MobileNetV2
- MobileNetV2 is fully connected layers but `lack a classifier` which is needed in Fall Detection task.
    + Add a fully connected layter *(to avoid information loss caused by rapid dimensionality reduction)* and a `softmax classifier` behind the original Framework.
- MobileNetV2 lack an attention mechanism
    + add CBAM (Convoluational Block Attention Module) mechanism at the beginning of the network
    + Improve learning ability of the shallow network for essential targets.

<div style="width: 100%; text-align: center;">
    <img style="" src="./imgs/modified_mobilenetv2.png" />
    <p>Modified MobileNetV2 Architecture</p>
</div>

## Framework Summarize

1. **Input preprocessed image into OpenPose network**
2. **Extract Features**
3. **Predict position confidence map and part affinity fields domain of the limbs, optimize network model parameters base on loss function**
4. **Associate the body parts with human body, marj image with the key points of the human body**

5. **Input iages with keypoints information into MobileNetV2, Extract features using deep convolution module**

6. **Adam algorithm to optimize model parameters according to loss function, finally detect Fall.**

## NOTES
### CNN Architectures
> Though fall detection as a classification problem. Using CNN architecture (Alex-net, VGG, Google-net, ResNet, PCAnet)

> Different CNN architecture means different in Number of NOdes, Layer Arrangement, Convolutional Layer size, Skip connections, Polling layer, ... => Each architecture is suitable for different problems.

- Some CNN architectures:
    + LeNet-5: ealiest CNN architectures => handwritten focus on digit recognition
    + AlexNet: Resurgence of deep learning
    + VGGNet: deep architecture with small 3x3 convolutional filters, for image classification.
    + GoogleLeNet: Inception architecture introduced the idea of using multiple filter sizes at each layer
    + ResNet: Residual Network , residual connection
    + `MobileNet`: Designed for mobile and embedded devcies. Optimized for efficiency and lightweight models.
    + ...

### Some Other Researches
Many researchers have explored the use of deep learning algorithms for feature extraction in fall detection. Some approaches utilized depth images but faced resolution limitations at greater distances from the camera. Another method combined multiple sensors and algorithms, but had restrictions related to lighting and small motions.

Researchers have successfully used CNN-based techniques like YOLOv5s and AlphaPose to extract features from video images for fall detection.

Currently, human pose estimation, such as the PPN (Pose Proposal Network) and OpenPose algorithms, is widely employed for fall detection.


### Keyword & Terms
- CBAM (Convolutional Block Attention Module): mechanism used in NN to help them pay more attention to important parts of data, such as: focusing on specific regions of image.
- Softmax function: is function that turns a set of numbers in to probabilities -> Making it easier to choose the most likely category in task such as image classification

