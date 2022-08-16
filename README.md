# The AnNOTEators Capstone Project
Greetings! This is our Summer 2022 Capstone Project for the Master of Applied Data Science at the University of Michigan School of Information. Our goal is to predict drum notes from audio to create sheet music. The team consists of Christopher Brown, Stanley Hung, and Severus Chang.  

# Introduction
Sheet music is a fundamental and important tool for most musicians. It makes individuals much faster and more efficient in preparing to play. Nowadays, obtaining properly written sheet music of a song could be troublesome unless that song is particularly popular and in the worst case a musician need to transcribe it themselves. The AnNOTEators project aims to help with this situation by leveraging neural networks to automatically transcibe each instrument part in a song. Due to the 8 week time limit for this project, the team decided to focus on transcribing drum notes and produce drum sheet music from a given song, rather than handle all instrument layers. You can find more details of the pipeline and framework in the [How this works](https://github.com/cb-42/siads_697_capstone_annoteators#how-this-works) section. We may expand the scope of this project to cover more instrument components in the future.

It is important to check out the [Known issues and limitations](https://github.com/cb-42/siads_697_capstone_annoteators#known-issues-and-limitations) sections for more information about factors to be aware of when using this package.

To learn more about the technical details of this project, please visit our [blog post]. **Attach link later**

# Software
Our requirements.txt file has a list of the Python library dependencies needed to run our Python scripts and Jupyter notebooks. Due to differing version dependencies in the model training and audio inference portions of the workflow, two environments are recommended.

A Docker image for audio input processing for inference can be acquired from the project [Docker Hub repository](https://hub.docker.com/r/cbrown42/annoteators_project). 

Note that 
- if you wish to use the Python `Spleeter` library for audio data preparation there are additional dependencies, such as `ffmpeg`, as noted [here](https://pypi.org/project/spleeter/).
- if you wish to get the pdf output from the pipline, [Musescore software](https://musescore.org/en/download) is required to be installed beforehand




# How does this work?

<img src="img/Flow_diagram.jpg" alt="Annoteators Flow Diagram" width="740">

For a more detailed explanation of each step, please visit our [blog post]. **Attach link later**

# Preparing audio input
The pipeline accept either local audio file (in various audio formats that supported by ffmpeg) or a Youtube link to the music video of your choice. Please note that...

- The pipeline expect a song that has drum part. 
- Higher audio input quality yields higher output quality. 
- Highly recommend using the original version of the song as an input to ensure noiseless signals and a clean signal. Other versions (e.g live performance) is not recommened


# Getting Started

There are a few ways to install and use the code, models, and environments we've developed.

### Interactive Web App
- tbd

### Docker image

Docker images for the model training and the audio inference environments can be acquired from our [Docker Hub repository](https://hub.docker.com/r/cbrown42/annoteators_project). These come with necessary Python libraries and other software pre-installed, such as MuseScore which is necessary for notation. You may wish to check out [getting started with Docker](https://docs.docker.com/get-started/#prepare-your-docker-environment) to learn more about set up.

For example, to pull the inference image, use the following command:  

```bash
docker pull cbrown42/annoteators_project:inference-0.02
```


### Command Line Interface

```bash
#please make sure you already have Github CLI installed
gh repo clone cb-42/siads_697_capstone_annoteators

#navigate to the root directory and install the necessary packages
pip install -r requirements.txt

#Below code will transcribe the audio from the youtube link below and output the pdf file in the root directory
#Musescore 3 need to be installed for pdf export
python main.py -i 'https://www.youtube.com/watch?v=hTWKbfoikeg' -o 'pdf'

```

### Notebook Enviornment
First download this repo or git clone this repo to your local computer

```bash
# Please make sure you already have Github CLI installed
gh repo clone cb-42/siads_697_capstone_annoteators
# Navigate to the root directory and install the necessary packages
pip install -r requirements.txt
```

Below is a quick demo code of transcribing a song to drum sheet music. Please refer to the `pipeline_demo` notebook (attach link later) for more details about parameters tuning 
```python

from inference.input_transform import drum_extraction, drum_to_frame, get_yt_audio
from inference.prediction import predict_drumhit
from inference.transcriber import drum_transcriber

# If you want to use the audio from a Youtube video...
path = get_yt_audio("Youtube link of your choice") 

# Or specify the file path to the audio file in your compauter
path = "the path to the audio file in your compauter"

# Extract drum track from the Audio File / Youtube Audio
drum_track, sample_rate = drum_extraction(path, kernel='demucs', mode='performance') 

# Create dataframe for prediction task
df, bpm = drum_to_frame(drum_track, sample_rate) 

#predict drum hit
prediction_df=predict_drumhit('inference/pretrained_models/annoteators/complete_network.h5', df, sample_rate)

#sheet music construction
song_duration = librosa.get_duration(drum_track, sr=sample_rate)
sheet_music = drum_transcriber(prediction_df, song_duration, bpm, sample_rate)

# Display in notebook env
sheet_music.sheet.show('text') # display in text format (musicxml protocol)
sheet_music.sheet.show() # display in png format 

#Eeport sheet music
sheet_music.sheet.write() # export in MusicXML format
sheet_music.sheet.write(fmt='musicxml.pdf') # export in pdf

```

# Custom training and pre-trained models (for Data Scientists)
We have uploaded various pre-trained models in the [pretrained_models folder](https://github.com/cb-42/siads_697_capstone_annoteators/tree/main/inference/pretrained_models/annoteators), differing primarily by augmentation strategy. The unaugmented base model is used in prediction by default. Each pre-trained model is a convolutional neural network (ConvNet) model that trained on the Expanded Groove MIDI Dataset (E-GMD) from the Google Magenta project. We also provided all the nessesary tooling for anyone that wishes to replicate or modify the training pipeline.  

## Source data  
This project used The Expanded Groove MIDI Dataset (E-GMD) for model development. E-GMD Dataset is a large dataset of human drum performances, with audio recordings annotated in MIDI. E-GMD contains 444 hours of audio from 43 drum kits and is an order of magnitude larger than similar datasets. It is also the first human-performed drum transcription dataset with annotations of velocity.

The E-GMD dataset was developed by a group of Google Researchers. For more information about the dataset, please visit their site: [The Expanded Groove MIDI Dataset](https://magenta.tensorflow.org/datasets/e-gmd).

## How were the data processed for model training? 
<img src="img/data_preparation.jpg" alt="Data Processing Diagram" width="740">

- Each drum track record in the dataset consist of 2 files - a MIDI file and a WAV audio file. The MIDI file and WAV file were synced to within 2ms time differences
- The WAV audio was sliced into a series of mini audio clips with the relevant label captured from the MIDI messages. 
- Each audio clip represents the sound of a single drum hit.
- Please refer to the `data_preparation.py` script for more details. We also prepared a notebook to showcase how data preparation elements work and connect together.

## Data Augmentation
Audio data augmentation can be applied to signals in the waveform or spectrogram domains, or both. We made several augmentation functions available in `augment_audio.py` and for convenienced these are also wrapped into the data preparation pipeline. We primarily explored and tested audio augmentations in the waveform space, though the base model trained on unaugmented audio ultimately performed best. Thus, we do not recommend augmentation for model development in this workflow.  
  
Augmentation can also be performed on the audio input used for inference. Depending on the kernel used for input signal preparation, we found that adding compression after processing resulted in better predictions. For more information, please see our [blog post]. **Attach link later**

## Model Architecture
- Serv to add

## Evaluation
- Serv to add

# Known issues and limitations
- The model has a poor performance in predicting multi-hit labels due to the lack of multi-hit labeled data in the training set. This could be fixed by modifying the data preparation algorithm.
- The quantization and time mapping algorithm may not be 100% accurate all the time. This approach is also very sensitive to the 'exact time' of each hit in the track. A slight delay (which always happens in human performed drumplay) sometimes could make the note duration detection go wrong. For example, a triplet note could be detected as a 16th note, due to a very little delay. A hidden markov chain model could be a solution to fix this problem - please visit our blog post for a deeper dive discussion on this.
- There is no standard style in writing drum sheet music. This project implemented a style of our choice, which may not suit everyone's styling preference. To change the notation style, it is necessary to modify the code in the transcriber script. This package uses the `Music21` package for sheet music construction.
- The standalone drum track demixed by `demucs` is not an original drum track. Some audio features could be altered or lost entirely during the demixing process. It is a known issue that the `demucs` processed drum track has a 'much cleaner signal' than the training drum track, which caused the prediction accuracy issue we observed. Please visit our blog post for a deeper dive and discussion about this, as well as the proposed methodology to fix this issue. 

# Future plans
- tbd
    
# Additional Resources
- tbd (relevant guides, tutorials, etc)

# References
    
## Software  
Our code uses the following open source packages and software:

**Python packages:**
- [Demucs](https://github.com/facebookresearch/demucs)
- [Librosa](https://github.com/librosa/librosa)
- [Mido](https://github.com/mido/mido/)
- [Music21](https://github.com/cuthbertLab/music21)
- [Pedalboard](https://github.com/spotify/pedalboard)
- [Spleeter](https://github.com/deezer/spleeter)
- [TensorFlow](https://github.com/tensorflow/tensorflow)    
    
**Notation software:**
- [MuseScore](https://musescore.org/en)
    
## Citations  
- tbd

# License
- tbd
