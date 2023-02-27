# Face Image Generation Through Adversarial Networks (GANs) 

## Introduction

Image acquisition systems have been largely deployed in recent years. Stronger computation power, machine learning architectures with higher accuracy, and the economic advancement in computer vision systems result in the expansion of the global computer vision market. In 2020, the global computer vision market generated 9.45 billion. Followed by a prediction of $41.1 billion by 2030 which implies a compound annual growth rate of 16%. As the computer vision market expands, a large quantity of images is now available, and the demand for synthesizing and processing images increases.

Generative Adversarial Networks (GANs) are deep neural network structures that contain two networks, pitting one network against the other, hence the name "adversarial". GANs have the potential to learn how to mimic any data distribution, so GANs can be taught to create worlds similar to ours in any domain, such as images, music, speech, and prose. In this project, we would use several types of unconditional GANs including DCGAN, StyleGAN2, and StyleGAN3 models to generate photorealistic face images and animated face images.

In this project, we used several types of unconditional GANs to generate photorealistic face images and animated face images. Models including DCGAN build from scratch,  pretrained StyleGAN2, and pretrained StyleGAN3 models. The proposed models were fine-tuned and then tested on the FFHQ Dataset and Anime Face Dataset. To compare the performances of the models, the training images were resized to the 64x64 resolution. The images generated by StyleGANs are more realistic as the generator architectures need more GPU hours to converge. StyleGAN3-r has the most robust performance since the network is equivariance to rotation and translation.


## Reference

For more information, please check out [the report paper written for this project](https://github.com/marswon0/face_image_generation/blob/main/assets/paper/Face%20Image%20Generation%20Through%20GANs.pdf).


### Tracking down the same portrait during training

<img src="/assets/images/training_history.JPG" width=450 height=150>

From portraits generated after 200 epochs (lower right) to 3200 epochs (upper left).

## Usage

### Preparing Dataset
    python dataset_tool.py --source=location_of_the_image_dataset --dest=file_destination --resolution=64x64
  
### Training

The training parameters used in this project were optimized for low resolution datasets. For training setup with higher resolution, check out the [StyleGAN3 Training configurations](https://github.com/NVlabs/stylegan3/blob/main/docs/configs.md).

    python train.py --outdir=file_destination --data=location_of_dataset --cfg=stylegan2 --gpus=1 --batch=32 --batch-gpu=32 --gamma=0.5 --mirror=1 --kimg=5000 --snap=10 --metrics=none --cbase=16384
  
<br/>  
  
    python train.py --outdir=file_destination --data=location_of_dataset --cfg=stylegan3-r --gpus=1 --batch=32 --batch-gpu=32 --gamma=1 --mirror=1 --kimg=2000 --snap=10 --metrics=none --cbase=16384
    
## Model Architectures

### DCGAN

GAN has two components: the generator and the discriminator. The **Generator(G)** generates synthetic data that mimic the training data. The **Discriminator(D)** identifies whether a photo is real or synthetic. During training, the generator aims to generate higher quality synthetic photos to trick the discriminator, whereas the discriminator keeps getting better at distinguishing between real and fake photos. The generator and discriminator architectures for DCGAN are shown below.

<img src="/assets/images/DCGAN.JPG" width=400 height=700>

### Loss Funciton

The loss function of a generative adversarial network can be presented as:

<img src="/assets/images/loss_func.JPG" width=650 height=30>

- x: data representation of an input image
- D(x): the possibility that the discriminator identifies the input is real
- z: a latent space vector sampled from the standard normal distribution
- G(z): the generator function mapping from the latent vector z to the data space
- D(G(z)): the probability that the output of generator G is a real image

### StyleGAN2

The StyleGAN is developed based on the [progressive GAN](https://arxiv.org/pdf/1710.10196.pdf). StyleGAN adapts progressive training method that uses bilinear sampling layers instead of nearest neighbor layers for upsampling. Compared with DCGAN, StyleGAN added a MLP between the latent space vector z and convolution layers. In the proposed DCGAN architecture, the latent space Z is constrained by the pre-defined gaussian distribution. GANs learn the underlying distribution of the real data through training. While gaussian distribution does not apply to the actual data, the DCGAN might have a poor performance. On the other hand, the intermediate latent space W is not limited by a single distribution. W can have any distribution mapped by the MLP network. Therefore, the GANs architecture with latent space W is more likely to learn the distribution of the real data.

<img src="/assets/images/generator_styleGAN.JPG" width=300 height=350>

To obtain the style vector y = (𝑦𝑠 , 𝑦𝑏), the affine transformation block A will be applied to the intermediate latent vector w. The style vector is transformed and incorporated in each block of the generator network through the AdaIN operation described in the equation below. 

<img src="/assets/images/adain.JPG" width=300 height=50>

In StyleGAN2 architecture, the AdaIN has been replaced with modulation and normalization described respectively in equation 3 and 4. **𝑤𝑖** and **𝑤𝑖′** are the original and modulated weight with the scaling factor **𝑠𝑖** for the corresponding feature map. The normalization is performed through the standard deviation **𝜎𝑗**. 

<img src="/assets/images/eq_norm.JPG" width=150 height=120>

Compared with the StyleGAN architecture, the additional noise B has been shifted outside the style block. There are two architectures available for the StyleGAN2 generator. In the revised architecture, the weights are adjusted with the style and normalization. For the weight demodulation architecture, the instance normalization is replaced by the demodulation operation **𝑤𝑖′j′k**.

<img src="/assets/images/eq_weight.JPG" width=250 height=80>

<img src="/assets/images/generator_stylegan2.JPG" width=450 height=350>

### StyleGAN3

StyleGAN2 has trouble with the texture sticking problem. Such problems might happen while having transitions such as changing facial expressions. During transition animation, textures such as hair, beard, and fur would stick to the screen rather than moving along with the generated face object. The texture sticking problem is mainly caused by unintentional positional references from the following sources: image borders, per-pixel noise inputs, positional encodings, and aliasing. These attributes provide the positional reference that results in the pixels generated by the network remaining unexpectedly on the same coordinates.

StyleGAN3 tackles the texture sticking problem encountered by StyleGAN2.  To eliminate all sources of positional reference, the generator architecture must be equivariant that implies operations such as ReLU should not insert any positional reference. Traditional neural networks operate in the discrete domain. Therefore, the feature map generated by convolution and ReLU would be a discrete feature map. This creates the problem of aliasing since the discrete feature map is considered as the result sampled from the underlying continuous feature map. StyleGAN3 replaced the learned 4x4x512 input constant layer in StyleGAN2 with a Fourier feature layer to emphasize and maintain the continuity of the input. The per-pixel noise inputs B are also removed since these input signals introduce unintentional positional references. The output skip connections are deleted; to compensate for this change, an Exponential Moving Average (EMA) operation is added to normalize the input before every convolution layer.

### Generator Architecture

<img src="/assets/images/generator_s3.JPG" width=400 height=350>

## Result

To test the generalization of the models, two datasets are used in this project: [the FFHQ Dataset](https://github.com/NVlabs/ffhq-dataset) and [the Anime Face Dataset](https://www.kaggle.com/datasets/splcher/animefacedataset). The images generated by StyleGANs are more realistic as the generator architectures need more GPU power to complete the training. StyleGAN3-r has the most robust performance since the network is equivariance to rotation and translation.

### DCGAN

<img src="/assets/images/result_dcgan.JPG">

### StyleGAN2

<img src="/assets/images/result_s2.JPG">

### StyleGAN3

<img src="/assets/images/result_s3.JPG">
