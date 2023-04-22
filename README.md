# MNIST Digit Generation using GAN 
Build and train a Generative Adversarial Networks (GAN) to generate fake MNIST digit images 

<img src="https://github.com/sssingh/mnist-digit-generation-gan/blob/master/assets/mnist_gan_title_pic.png?raw=true" width="800" height="400">

## Features
⚡Image Generation  
⚡Generative Adverserial Network (GAN)  
⚡Fully Connected Neural Network Layers  
⚡MNIST  
⚡PyTorch  

## Table of Contents
- [Introduction](#introduction) 
- [Objective](#objective)
- [Dataset](#dataset)
- [Solution Approach](#solution-approach)
- [How To Use](#how-to-use)
- [License](#license)
- [Get in touch](#get-in-touch)


## Introduction
The Generative Adversarial Networks (GAN) are a special neural network where two networks compete to maximize their gain (i.e., minimize their losses in machine-learning lingo). GAN's objective is to generate the _synthetic_ or _fake_ data based on the training data it learns from, hence a `Generative` network. Since two networks are _fighting_ with each other, the network is called `Adversarial.` A simple GAN is composed of a dataset (MNIST in our case) and two neural networks. A `Generator` network takes a random `noise` vector and learns to transform this noise such that its probability distribution eventually comes very close to the data it's learning from. A `Discriminator` network is a neural network-based classifier. Both `Generator` and `Discriminator` train in parallel. `Discriminator` gets to see images from actual MNIST dataset as well _fake_ pictures generated by the `Generator,` and its job is to correctly classify _real_ images vs. _fake_ images. At the same time, based on how `Discriminator` is performing in real vs. fake classification, `Generator` keeps improving its generated images to make them look like the images taken from the real dataset. Basically, `Generator` tries to fool the `Discriminator` and `Discriminator` tries not to get fooled.

A typical GAN network is shown below...<br><br>

<img src="https://github.com/sssingh/mnist-digit-generation-gan/blob/master/assets/simple_GAN.png?raw=true" width="800">

Have a look at [GAN Original Paper](https://www.researchgate.net/publication/263012109_Generative_Adversarial_Networks) for more details<br>. In addition, there are some exciting applications of GANs one can explore...
* [StackGAN](https://arxiv.org/abs/1612.03242): Generates 256x256 photo-realistic images conditioned on text descriptions. For example, if a description of a bird is provided, it will be able to generate an image of the bird based on the description; this will be an imaginary picture of a bird that does not exist in real words but looks real.
* [iGAN](https://github.com/junyanz/iGAN): Based on user-drawn simple sketches, iGAN could produce photo-realistic samples that best matches the user sketch in real-time.
* [CartoonGAN](https://video.udacity-data.com/topher/2018/November/5bea23cd_cartoongan/cartoongan.pdf): CartoonGAN can transform the photos of real-world scenes into cartoon style images. For example, it can take an image of a real person's face and produce a _catoonized_ image of that face.
* [CycleGAN](https://github.com/junyanz/CycleGAN):  CycleGAN performs an image-to-image translation. Model trains in an `unsupervised` manner using a collection of images from the source and target domain that do not need to be related in any way. Models trains without any paired examples. For example, it can take a photo of a summer landscape and transform the same image into a winter landscape.

## Objective
Our goal in this project is to build a GAN and train it over the MNIST dataset so that the network learns to generate _fake_ MNIST digit images that would have come from the real MNIST dataset.

## Dataset
- Dataset consists of 60,000 training images and 10,000 testing images. For this task, we'll use the training dataset.
Every image in the dataset will belong to one of the ten classes (digit 0 to 9); however, the image labels do not matter for this task, and we won't use them.
- Each image in the dataset is a 28x28 pixel grayscale image, a zoomed-in single image shown below...

<img src="https://github.com/sssingh/mnist-digit-generation-gan/blob/master/assets/mnist_single_image.png?raw=true">


Few samples of _real_ MNIST images are shown below...

<img src="https://github.com/sssingh/mnist-digit-generation-gan/blob/master/assets/mnist_samples.png?raw=true" width="800">

We will use the in-built MNIST dataset from PyTorch's `torchvision .` it's a clean, pre-processed dataset that pairs the image and respective label nicely; labels are not required for the task at hand; they will be ignored. Alternatively, the raw dataset can be downloaded from the original source [here](http://yann.lecun.com/exdb/mnist/). The raw dataset comes as a set of zip files containing training images, training images, testing images, and testing images in separate files. 

## Solution Approach
### Data Load
- Data is downloaded using the `torchvision` dataset. 
- The training dataset is then wrapped in a dataloader object with a `batch_size` of 64. We discard the testing dataset. Note that even though the dataloader will give us the images associated, we'll simply ignore them.

### Network Definition
- First, we define the `Discriminator` network 
    - Its a typical binary classifier where it'd accept 784 (28x28) inputs and produces a single `logit` output that's used to classify the input image as _real_ (1) or _fake_ (0)
    - Network has four fully-connected `Linear` layers with `LeakyReLU` activation having a `negative-slope` of 0.2. Furthermore, a `dropout` of 30% is applied after each linear layer except the last one. The activation function and negative slope value are based on the GAN paper recommendation.
    - The `forward` method of the Discriminator flattens the input it receives to make it a tensor of shape (batch_size, 784); this is then passed through the network.
- Then, we define the `Generator` network
    - Generator will consume a `noise` vector (z of length 100) and up-sample it by passing it through various layers of the network.
    - Network has four fully-connected `Linear` layers with `LeakyReLU` activation having a `negative-slope` of 0.2. A `dropout` of 30% is applied after each linear layer except the last one
    The final linear layer output is then passed through the `tanh` function to produce the final output of the Generator between -1 and 1. This is again as per the original GAN paper.

The complete structure of the `Discriminator` and `Generator` network are shown below...

<img src="https://github.com/sssingh/mnist-digit-generation-gan/blob/master/assets/network.png?raw=true" width="800"><br><br>

Once we have both `Discriminator` and `Generator` networks ready, they are assembled to build the MNIST GAN as shown below..

<img src="https://github.com/sssingh/mnist-digit-generation-gan/blob/master/assets/mnist_GAN.png?raw=true" width="800"><br><br>

### Loss Definition
GAN training is a bit different compared to our typical supervised neural-network training. In the case of GAN, two separate networks are being trained together, and this network has different and opposing objectives (i.e., they are competing). The `Discriminator` is trying to identify if the image sample is _real_ and from the actual MNIST dataset or its  _fake_ images generated by our `Generator.` Note that we are NOT interested incorrectly classifying the digits themselves. We'd need to define two separate loss functions.
1. real_loss: calculates loss when images are drawn from the actual dataset. The predicted output is compared against the target label `1`, indicating real images.
2. fake_loss: calculates loss when images are generated by the Generator. The predicted output is compared against the target label `0`, indicating fake images. 

`Discriminator` computes both of the above losses and adds them together to get a `total-loss` for back-propagation
`Generator` computes the `real_loss` to check its success in _fooling_ the `Discriminator.` In other words, even though it generates fake images (target 0), by computing real_loss, it compares Discriminator's output with `1`. In effect, generator loss has its labels `flipped.`

### Network Training
Since we are training two separate networks, we need two separate optimizers for each network. In both cases, we use `Adam` optimizer with a `learning-rate` of `0.002.`
- Since classification is between two classes (real and fake) and our Discriminator outputs a `logit,` we use `BCEWithLogitsLoss` as the loss function. This function internally first applies a `sigmoid` activation to logits and then calculates the loss using the `BCELoss` (log loss) function.
Before starting training, we create a (16 x 100) `fixed-random-noise-vector` drawn from a `normal distribution` between range `-1 and 1`. This vector is kept fixed throughout the training. After each epoch of training, we feed the noise vector to, so far, trained Generator to generate _fake_ images; these images help us visualize how and if generated image quality is improving or not. A sample of the noise vector is shown below..

<img src="https://github.com/sssingh/mnist-digit-generation-gan/blob/master/assets/latent_vector.png?raw=true" width="800"><br><br>

- `Discriminator` is trained as follows...
    - A batch of `real` MNIST images are drawn from the dataloader
    - Each image in the batch is then `scaled` to values between `-1 and 1`. This is a crucial step and required because Discriminator looks at _real_ images from MNIST dataset and looks at _fake_ images from Generator whose output is in range `-1 to 1` (last layer output Generator network is `tanh` activated). So we need to ensure that the range of input values is consistent in both cases.
    - data batch is then fed to Discriminator, its predicted output is captured, and `real_loss` is calculated
    - A batch noise data (`z`) drawn from a `normal distribution` between range `-1 and 1` is created
    - Noise `z` is then fed through the Generator, its outputs (fake images) are captured, and `fake_loss` is calculated
    - Then discriminator's `total_loss` is computed as `real_loss + fake_loss`
    - Finally, `total_loss` is back-propagated using Discriminator's optimizer
- After one batch of `Discriminator` training (above), the `Generator` is trained as follows...
    - A batch of noise data (`z`) drawn from a `normal distribution` between range `-1 and 1` is created
    - Noise `z` is then fed through the Generator, its outputs (fake images) are captured
    - The generated _fake_ images are then fed through the `Discriminator,` and its predicted output is captured, and `real_loss` is calculated
    - Note that for _fake_ generated images we are calculating `real_loss` (and not fake_loss) as discussed in [Loss Definition](#loss-definition) section above 
    - Above computed loss is then back-propagated using Generator's optimizer
- At the end of each epoch, the `fixed-random-noise-vector` is fed to the trained Generator to produce a batch of _fake_ images; we then save these images as (`fixed_samples.pkl`). We can load and view these protected images later for further analysis.
After training our GAN network for `100` epochs, we plot both Generator and Discriminator losses and it looks like this...

<img src="https://github.com/sssingh/mnist-digit-generation-gan/blob/master/assets/losses.png?raw=true" width="800"><br><br>

The above plot does not look like a typical neural-network training loss plot. There are huge fluctuations at the beginning, and it's very wobbly after that. This behavior is very typical of GAN training and expected. At the end of the 100th epoch, the discriminator loss (in red) seems to be going down, and generator loss (in blue) increases. It's possible that if we train our network for even more epochs, losses may converge, indicating an equilibrium between the competing networks. `Equilibrium` is the fundamental idea behind GAN, which is drawn from  [The Game Theory](https://web.archive.org/web/20100610071152/http://www.ewp.rpi.edu/hartford/~stoddj/BE/IntroGameT.htm) that suggests that competing rationale agents will ultimately reach an equilibrium where they can't improve anymore.

### Visualize Training Progress
Let's visualize the intermediate generator outputs that we saved during the training. This will show us how Generator learns and generates better fake images as training progresses.

Generator output after `1 epoch` of training...

<img src="https://github.com/sssingh/mnist-digit-generation-gan/blob/master/assets/generated_images_epoch_1.png?raw=true" width="800"><br><br>

Generator output after '10 epochs` of training...

<img src="https://github.com/sssingh/mnist-digit-generation-gan/blob/master/assets/generated_images_epoch_10.png?raw=true" width="800"><br><br>

Generator output after '50 epochs` of training...

<img src="https://github.com/sssingh/mnist-digit-generation-gan/blob/master/assets/generated_images_epoch_50.png?raw=true" width="800"><br><br>

Generator output after `100 epochs` of training...

<img src="https://github.com/sssingh/mnist-digit-generation-gan/blob/master/assets/generated_images_epoch_100.png?raw=true" width="800"><br><br>

We can see how Generator has improved from generating random noisy blobs after 1st epoch to realistic-looking MNIST digits after 100 epochs. 

## How To Use
1. Ensure the below-listed packages are installed
    - `NumPy`
    - `pickle`
    - `matplotlib`
    - `torch`
    - `torchvision`
2. Download `mnist_gan.ipynb` jupyter notebook from this repo
3. Execute the notebook from start to finish in one go. If a GPU is available (recommended), it'll use it automatically; otherwise, it'll fall back to the CPU. 
4. A machine with `NVIDIA Quadro P5000` GPU with 16GB memory takes approximately 20 minutes to train for 100 epochs.
5. Longer training will yield better results
7. A trained model can be used to generate fake MNIST digits, as shown below...

```python
    # Bring generator back to cpu and set eval mode on
    g.to('cpu')
    g.eval()
    # Feed a latent vecor of size 100 to trained generator and get a fake generated image back
    z = np.random.uniform(-1, 1, size=(1, 100))
    z = torch.from_numpy(z).float()
    fake_image = g(z)
    # Reshape and display
    fake_image = fake_image.view(1, 1, 28, 28).detach()
    display_images(fake_image, n_cols=1, figsize=(2, 2))
```

A randomly generated fake image using the above code is shown below; we can see that the generated _fake_ image looks very similar to the actual number `3` image shown [above](#dataset)...

<img src="https://github.com/sssingh/mnist-digit-generation-gan/blob/master/assets/test_result.png?raw=true">

## License
[![MIT License](https://img.shields.io/badge/License-MIT-green.svg)](https://choosealicense.com/licenses/mit/)

## Get in touch
[![website](https://img.shields.io/badge/web_site-8B5BE8?style=for-the-badge&logo=ko-fi&logoColor=white)](https://www.datamatrix-ml.com)
[![twitter](https://img.shields.io/badge/twitter-1DA1F2?style=for-the-badge&logo=twitter&logoColor=white)](https://twitter.com/@thesssingh)
[![linkedin](https://img.shields.io/badge/linkedin-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/sssingh/)

[Back To The Top](#MNIST-Digit-Generation-using-GAN)
