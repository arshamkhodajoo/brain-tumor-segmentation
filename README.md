<img src="https://github.com/arshamkhodajoo/tumor-export-frontend/raw/main/public/review.png" />

[![made-with-python](https://img.shields.io/badge/Made%20with-Python-1f425f.svg)](https://www.python.org/)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)
[![Imports: isort](https://img.shields.io/badge/%20imports-isort-%231674b1?style=flat&labelColor=ef8336)](https://pycqa.github.io/isort/)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)


# Table of content
* [The Project: Problem, Solution and Product](#about-tumor-expert)
* [Toturial](#toturial)
* [Install using Docker](#docker-installation)
* [Install Manually](#manual-installation)
* [Deep Learning Model](#deep-learning-model)
* [Using API interface](#api)

## About Tumor Expert
### Problem
Brain tumors include the most threatening types of tumors around the world. Glioma, the most common primary brain tumors, occurs due to the carcinogenesis of glial cells in the spinal cord and brain. Glioma is characterized by several histological and malignancy grades, and an average survival time of fewer than 14 months after diagnosis for glioblastoma patients.

Magnetic Resonance Imaging (MRI), a popular non-invasive strategy, produces a large and diverse number of tissue contrasts in each imaging modality and has been widely used by medical specialists to diagnose brain tumors.

However, the manual segmentation and analysis of structural MRI images of brain tumors is an arduous and time-consuming task which, thus far, can only be accomplished by professional neuroradiologists

### Solution
Segmenting brain tumors and subregions automatically from multi-modal MRI is important for reproducible and accurate measurement of the tumors, and this can assist better diagnosis, treatment planning and evaluation.

Recently, deep learning methods with Convolutional Neural Networks (CNNs) have become the state-of-the-art approaches for brain tumor segmentation.
A key problem for CNN-based segmentation is to design a suitable network structure and training strategy. Using a 2D CNN in a slice-by-slice manner has a relatively low memory requirement, but the network ignores 3D information, which will ultimately limit the performance of the segmentation. Using 3D CNNs can better exploit 3D features.


### The Product
We introduce Tumor Expert, web-based application which utilize 3D CNNs (or 3D U-net specifically) for automatic segmenting brain tumors (Glioma) while enable you to visualize slices and modalities with tumor area spotted with tree label:

Label  | Region color
------------- | -------------
Whole Tumor  | Blue
Tumor Core  | Red
Enhancing Tumor | Green

## Toturial
### Upload Files
<img src="https://github.com/arshamkhodajoo/tumor-expert-frontend/blob/main/public/upload-guid.gif" width="300px"/>

- Application expects four MRI sequences (T1, T2, T1CE, FLAIR), and in order to classify files, corresponding keyword should be placed in each filename

Weight  | Example filename
------------- | -------------
T1  | `example_sample_somthing_t1.nii.gz`
T1CE  | `example_sample_somthing_t1ce.nii.gz`
T2 | `example_sample_somthing_t2.nii.gz`
FLAIR | `example_sample_somthing_flair.nii.gz`

you can find sample files [here](https://github.com/arshamkhodajoo/tumor-expert/tree/main/public/samples).

### View, Scale, Segment
<img src="https://github.com/arshamkhodajoo/tumor-expert-frontend/blob/main/public/view-expert.gif" width="300px">

- View options are on the top left corner, you can switch view from sequence weights, hide segmentation and scale image on screen.
- There is also static information about segmentation colors on top left corner.
- The Slicer is on the bottom.

## Installation and Setup

### Docker Installation
build The Container:

`docker build -t brain-tumor-expert <path_to>/tumor-expert`

Run The Container

`docker run -dp 8000:8000 brain-tumor-expert`

### Manual installation
Tumor Expert uses React for Web applications which placed in another repository, thus we need to clone and get the production build first:
```
git clone https://github.com/arshamkhodajoo/tumor-expert-frontend.git
cd tumor-expert-frontend
```

now install dependencies and build app
```
npm install
npm run build
```

At this point, you will have `build/` inside the app folder,
and we will set the ENV variable to the `build/` directory path. We will use it to serve HTML and static files from the fast-API end.
#### Note: run this command only when your current directory is `<path>/tumor-expert-frontend`

for Linux: 

`export REACT_BUILD="$(pwd)/build"`

for PowerShell (Windows): 


```
$ENV:REACT_BUILD=pwd
$ENV:REACT_BUILD += "\build"
```

At this point, we are ready to setup Server.
get out of `/tumor-expert-frontend` and run:

```
git clone https://github.com/arshamkhodajoo/tumor-expert.git
cd tumor-expert
```

install dependencies:

`pip install -r requirements.txt`

install source as local package:
`pip install -e .`

set trained model path to onnx file place in repositiry:

for Linux:

`export WEIGHTS="$(pwd)/public/brain-tumor-segmentation-0002/brain-tumor-segmentation-0002.onnx"`

for PowerShell (Windows)

```
$ENV:WEIGHTS=pwd
$ENV:WEIGHTS += "/public/brain-tumor-segmentation-0002/brain-tumor-segmentation-0002.onnx"
```

<italic>windows truly sucks, I feel it with each of my nerve cells.</italic>

Run Server:

```
cd bras/app/
uvicorn app:app --port 8000
```

now server is running on `localhost:8000`

open your browser and type `localhost:8000`to load application.

## Deep Learning Model
### Brief introduction
**U-Net** architecture (shown in the Fig below) is characterised by a symmetric
U-shape, and can be divided into two parts, i.e., encoder and decoder. The first
part is the contracting path (encoder) which is transforming the input volume
into lower dimensional space. The encoder has a modular structure consisting
of repeating convolution blocks. Each block has two smaller blocks of transformations (dark and light blue blocks on the Fig. 2). The first smaller block is
reducing the spatial dimensions of the input feature map by a factor of two
via convolutional layer with kernels 3x3x3 and stride 2x2x2, then instance normalization and Leaky ReLU activation with negative slope if 0.01 are applied
(dark blue block). Next feature map is transformed with almost the same set of
operations except that the convolutional layer has stride 1x1x1 (light blue).

<img src="https://github.com/arshamkhodajoo/tumor-expert-frontend/blob/main/public/structure.png" />

### Inference and Production
[OpenVino's runtime](https://docs.openvino.ai/latest/index.html) been used to accelerate processing and optimize memory usage at production.</br>
there is limited list of supported Intel CPUs than could benefit from this inference optimization,[ see this](https://www.intel.com/content/www/us/en/developer/tools/openvino-toolkit/overview.html) for more information

For more documentation of model Inputs and Outputs, [check this link](https://github.com/openvinotoolkit/open_model_zoo/tree/master/models/public/brain-tumor-segmentation-0002).

### Custome Train
To train locally, please check `notebooks/colab_brats_train.ipynb` for complete training/evaluating pipline, also `config/unet.yaml` for model structure and hyperparameter.

model hyperparameters were taken from this [paper](https://arxiv.org/abs/2110.03352)

 **You may experience Runtime Reset and crashes when using Google Colab free version, consider using Colab Pro for more CUDA memory**

## API 
API Interface also can be used separately, you can use inference call at `localhost:8000/view` to get segmentation nifti file.
all four modals (t1, t1ce, t2, flair) nifti files should be sent as multi-part form data.

example using JavaScript and Axios:
```javascript
const data = {
  t1: File(...),
  t1ce: File(...),
  t2: File(...),
  flair: File(...)
}

const form = new Form()
for (const [key, value] of Object.entries(data)) {
        form.append(key, value)
}

axios({
        method: "POST",
        url: "localhost:8000/views/",
        data: form,
        headers: {
            "Content-Type": "multipart/form-data"
        },
        responseType: 'blob'
    })
```
