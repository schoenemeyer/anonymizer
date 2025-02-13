# Anonymizer - using NVIDIA GPUS and NVIDIA NGC Tensorflow container

This is an fork from understand-ai. It works similar with 2 small changes. To get the best performance on NVIDIA GPUs, we are using an NGC Tensorflow container.

To make it easy for everyone to use these weights in their own projects the model is trained with 
[Tensorflow Object Detection API](https://github.com/tensorflow/models/tree/master/research/object_detection).

## Disclaimer

Note that the version here is not identical to the anonymizer we use with our customers. The models are fairly similar,
but the glue-code is written for easy-of-use instead of speed.  
For this reason no multiprocessing code or batched detection and blurring are used in this repository.

This version of the anonymizer is trained to detect faces and license plates in images recorded with sensors that are
typically used in autonomous vehicles. It will not work on low-quality or grayscale images and will also not work on 
fish-eye or other extreme camera  configuration.
If there is high demand for models specialised for certain camera configurations, we might decide to open-source our 
more specialised models as well.


## Examples

![License Plate Example Raw](images/coco02.jpg?raw=true "Title")
![License Plate Anonymized](images/coco02_anonymized.jpg?raw=true "Title")

![Face Example Raw](images/coco01.jpg?raw=true "Title")
![Face Example Anonymized](images/coco01_anonymized.jpg?raw=true "Title")


## Installation

To install the anonymizer just clone this repository, pull the

```bash
docker pull nvcr.io/nvidia/tensorflow:20.11-tf1-py3 
```

This container comes with the necessary python3.6 environment. One can see the full details of this release https://docs.nvidia.com/deeplearning/frameworks/tensorflow-release-notes/rel_20-11.html#rel_20-11 

We assume, the docker engine with the NVIDIA runtime on your GPU system is already installed. If not please visit our guide how to install
https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker

The sequence of commands to do all this is

```bash
docker run -it --gpus all  --rm -v `pwd`:`pwd` -w `pwd`  nvcr.io/nvidia/tensorflow:20.11-tf1-py3
git clone https://github.com/schoenemeyer/anonymizer
cd anonymizer
```
## These are the modifications from the master version in understand-ai , a) requirements.txt and b) docker run

```bash
cat requirements.txt
pytest
flake8
numpy
Pillow
requests
googledrivedownloader
tqdm
```

```bash
docker run -it --gpus all  --rm -v `pwd`:`pwd` -w `pwd`  nvcr.io/nvidia/tensorflow:20.11-tf1-py3
pip install --upgrade pip
pip install -r requirements.txt
```

To make sure everything is working as intended run the test suite with the following command

```bash
pytest
```

Running the test cases can take several minutes and is dependent on your GPU and internet speed.  
Some test cases download model weights and some perform inference to make sure everything works as intended.


## Usage

Since the weights will be downloaded automatically all that is needed to anonymize images is to run

```bash
PYTHONPATH=$PYTHONPATH:. python anonymizer/bin/anonymize.py --input /path/to/input_folder --image-output /path/to/output_folder --weights /path/to/store/weights
```

from the top folder of this repository. This will save both anonymized images and detection results as json-files to
the output folder.

### Advanced Usage

In case you do not want to save the detections to json, add the parameter `no-write-detections`.
Example:

```bash
PYTHONPATH=$PYTHONPATH:. python anonymizer/bin/anonymize.py --input /path/to/input_folder --image-output /path/to/output_folder --weights /path/to/store/weights --no-write-detections
```

Detection threshold for faces and license plates can be passed as additional parameters.
Both are floats in [0.001, 1.0]. Example:

```bash
PYTHONPATH=$PYTHONPATH:. python anonymizer/bin/anonymize.py --input /path/to/input_folder --image-output /path/to/output_folder --weights /path/to/store/weights --face-threshold=0.1 --plate-threshold=0.9
```

By default only `*.jpg` and `*.png` files are anonymized. To for instance only anonymize jpgs and tiffs, 
the parameter `image-extensions` can be used. Example:

```bash
PYTHONPATH=$PYTHONPATH:. python anonymizer/bin/anonymize.py --input /path/to/input_folder --image-output /path/to/output_folder --weights /path/to/store/weights --image-extensions=jpg,tiff
```

The parameters for the blurring can be changed as well. For this the parameter `obfuscation-kernel` is used.
It consists of three values: The size of the gaussian kernel used for blurring, it's standard deviation and the size
of another kernel that is used to make the transition between blurred and non-blurred regions smoother.
Example usage:

```bash
PYTHONPATH=$PYTHONPATH:. python anonymizer/bin/anonymize.py --input /path/to/input_folder --image-output /path/to/output_folder --weights /path/to/store/weights --obfuscation-kernel="65,3,19"
```

## Attributions

An image for one of the test cases was taken from the COCO dataset.  
The pictures in this README are under an [Attribution 4.0 International](https://creativecommons.org/licenses/by/4.0/legalcode) license.
You can find the pictures [here](http://farm4.staticflickr.com/3081/2289618559_2daf30a365_z.jpg) and [here](http://farm8.staticflickr.com/7062/6802736606_ed325d0452_z.jpg).
