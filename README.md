[//]: # (Image References)

[image1]: ./images/pipeline.png "ASR Pipeline"
[image2]: ./images/select_kernel.png "select nlp-vui kernel"

# Introduction

In this project, I built multiple deep neural networks (...) that can function as part of an end-to-end Automatic Speech Recognition (ASR) pipeline. This pipeline accepts raw audio as input, then maps these audio features to transcribed text and returns text.

The pipeline consists of:

- Feature Extraction
- Acoustic Model
- Decoder

![ASR Pipeline][image1]

We begin by investigating the LibriSpeech dataset that will be used to train and evaluate your models. Your algorithm will first convert any raw audio to feature representations that are commonly used for ASR. You will then move on to building neural networks that can map these audio features to transcribed text. After learning about the basic types of layers that are often used for deep learning-based approaches to ASR, you will engage in your own investigations by creating and testing your own state-of-the-art models. Throughout the notebook, we provide recommended research papers for additional reading and links to GitHub repositories with interesting implementations.

# Setup

This project requires GPU acceleration to run efficiently.

## Local Machine (Option)

If you are planning to run on a local machine, I recommend only doing this option if you have a powerful GPU meant for deep learning. For instance, [ASUS - ROG GU502GV 15.6" Gaming Laptop - Intel Core i7 - 16GB Memory - NVIDIA GeForce RTX 2060 - 1TB SSD + Optane - Brushed Metallic Black](https://www.bestbuy.com/site/asus-rog-gu502gv-15-6-gaming-laptop-intel-core-i7-16gb-memory-nvidia-geforce-rtx-2060-1tb-ssd-optane-brushed-metallic-black/6338248.p?skuId=6338248).

You should run this project with GPU acceleration for best performance.

1. Clone the repository, and navigate to the downloaded folder.
```
git clone hhttps://github.com/james94/Capstone-VUI-Speech-Recognizer-NLPnd
cdCapstone-VUI-Speech-Recognizer-NLPnd
```

2. Create (and activate) a new environment with Python 3.6 and the `numpy` package.

	- __Linux__ or __Mac__: 
	```
	conda create --name nlp-vui python=3.5 numpy
	source activate nlp-vui
	```
	- __Windows__: 
	```
	conda create --name nlp-vui python=3.5 numpy scipy
	activate nlp-vui
	```

3. Install a few pip packages.
```
pip install -r requirements.txt
```

4. Switch [Keras backend](https://keras.io/backend/) to TensorFlow.
	- __Linux__ or __Mac__: 
	```
	KERAS_BACKEND=tensorflow python -c "from keras import backend"
	```
	- __Windows__: 
	```
	set KERAS_BACKEND=tensorflow
	python -c "from keras import backend"
	```
	- __NOTE:__ a Keras/Windows bug may give this error after the first epoch of training model 0: `‘rawunicodeescape’ codec can’t decode bytes in position 54-55: truncated \uXXXX `. 
To fix it: 
		- Find the file `keras/utils/generic_utils.py` that you are using for the capstone project. It should be in your environment under `Lib/site-packages` . This may vary, but if using miniconda, for example, it might be located at `C:/Users/username/Miniconda3/envs/nlp-vui/Lib/site-packages/keras/utils`.
		- Copy `generic_utils.py` to `OLDgeneric_utils.py` just in case you need to restore it.
		- Open the `generic_utils.py` file and change this code line:</br>`marshal.dumps(func.code).decode(‘raw_unicode_escape’)`</br>to this code line:</br>`marshal.dumps(func.code).replace(b’\’,b’/’).decode(‘raw_unicode_escape’)`

6. Obtain the `libav` package.
	- __Linux__: `sudo apt-get install libav-tools`
	- __Mac__: `brew install libav`
	- __Windows__: Browse to the [Libav website](https://libav.org/download/)
		- Scroll down to "Windows Nightly and Release Builds" and click on the appropriate link for your system (32-bit or 64-bit).
		- Click `nightly-gpl`.
		- Download most recent archive file.
		- Extract the file.  Move the `usr` directory to your C: drive.
		- Go back to your terminal window from above.
	```
	rename C:\usr avconv
    set PATH=C:\avconv\bin;%PATH%
	```

~~~bash
# Install librosa
conda install -y -c conda-forge librosa
conda install -y -c conda-forge av

# Install avconv on Ubuntu 18.04 WSL
# https://askubuntu.com/questions/1098407/how-do-you-install-avconv-on-ubuntu-18-04
sudo apt-get install ffmpeg
cd /usr/bin && sudo ln ffmpeg avconv && sudo ln ffmpeg avprobe
~~~


7. Obtain the appropriate subsets of the LibriSpeech dataset, and convert all flac files to wav format.
	- __Linux__ or __Mac__: 
	```
	wget http://www.openslr.org/resources/12/dev-clean.tar.gz
	tar -xzvf dev-clean.tar.gz
	wget http://www.openslr.org/resources/12/test-clean.tar.gz
	tar -xzvf test-clean.tar.gz
	mv flac_to_wav.sh LibriSpeech
	cd LibriSpeech
	./flac_to_wav.sh
	```
	- __Windows__: Download two files ([file 1](http://www.openslr.org/resources/12/dev-clean.tar.gz) and [file 2](http://www.openslr.org/resources/12/test-clean.tar.gz)) via browser and save in the `Capstone-VUI-Speech-Recognizer-NLPnd` directory.  Extract them with an application that is compatible with `tar` and `gz` such as [7-zip](http://www.7-zip.org/) or [WinZip](http://www.winzip.com/). Convert the files from your terminal window.
	```
	move flac_to_wav.sh LibriSpeech
	cd LibriSpeech
	powershell ./flac_to_wav.sh
	```

8. Create JSON files corresponding to the train and validation datasets.
```
cd ..
python create_desc_json.py LibriSpeech/dev-clean/ train_corpus.json
python create_desc_json.py LibriSpeech/test-clean/ valid_corpus.json
```

9. Create an [IPython kernel](http://ipython.readthedocs.io/en/stable/install/kernel_install.html) for the `nlp-vui` environment.  Open the notebook.
```
python -m ipykernel install --user --name nlp-vui --display-name "nlp-vui"
jupyter notebook vui_notebook.ipynb
```

10. Before running code, change the kernel to match the `nlp-vui` environment by using the drop-down menu.  Then, follow the instructions in the notebook.

__NOTE:__ While some code has already been implemented to get you started, you will need to implement additional functionality to successfully answer all of the questions included in the notebook. __Unless requested, do not modify code that has already been included.__


## Amazon Web Services (Option)

Launch a GPU EC2 instance. For instance, you can choose to launch [AWS Deep Learning AMI (Ubuntu 18.04)](https://aws.amazon.com/marketplace/pp/Amazon-Web-Services-AWS-Deep-Learning-AMI-Ubuntu-1/B07Y43P7X5) as a GPU instance.

1. Follow the Cloud Computing Setup instructions lesson to create an EC2 instance. (The lesson includes all the required package and library installation instructions.)

2. Obtain the appropriate subsets of the LibriSpeech dataset, and convert all flac files to wav format.
```
wget http://www.openslr.org/resources/12/dev-clean.tar.gz
tar -xzvf dev-clean.tar.gz
wget http://www.openslr.org/resources/12/test-clean.tar.gz
tar -xzvf test-clean.tar.gz
mv flac_to_wav.sh LibriSpeech
cd LibriSpeech
./flac_to_wav.sh
```

3. Create JSON files corresponding to the train and validation datasets.
```
cd ..
python create_desc_json.py LibriSpeech/dev-clean/ train_corpus.json
python create_desc_json.py LibriSpeech/test-clean/ valid_corpus.json
```

4. Start Jupyter:
```
jupyter notebook --ip=0.0.0.0 --no-browser
```

5. Look at the output in the window, and find the line that looks like: `http://0.0.0.0:8888/?token=3156e...` Copy and paste the **complete** URL into the address bar of a web browser (Firefox, Safari, Chrome, etc). Before navigating to the URL, replace 0.0.0.0 in the URL with the "IPv4 Public IP" address from the EC2 Dashboard.


<!-- # Automatic Speech Recogition Pipeline

## Feature Extraction

## Acoustic Model

## Decoder

# Future Enhancements

# Resources -->

## Suggestions to Make your Project Stand Out!

#### (1) Add a Language Model to the Decoder

The performance of the decoding step can be greatly enhanced by incorporating a language model.  Build your own language model from scratch, or leverage a repository or toolkit that you find online to improve your predictions.

#### (2) Train on Bigger Data

In the project, you used some of the smaller downloads from the LibriSpeech corpus.  Try training your model on some larger datasets - instead of using `dev-clean.tar.gz`, download one of the larger training sets on the [website](http://www.openslr.org/12/).

#### (3) Try out Different Audio Features

In this project, you had the choice to use _either_ spectrogram or MFCC features.  Take the time to test the performance of _both_ of these features.  For a special challenge, train a network that uses raw audio waveforms!

## Special Thanks

We have borrowed the `create_desc_json.py` and `flac_to_wav.sh` files from the [ba-dls-deepspeech](https://github.com/baidu-research/ba-dls-deepspeech) repository, along with some functions used to generate spectrograms.
