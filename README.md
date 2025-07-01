# PLCTestbench

PLCTestbench is a companion tool for researchers and developers working on Packet Loss Concealment (PLC). It greatly simplifies the process of measuring the reconstruction quality of PLC algorithms and allows to easily test the effects of different packet loss models and distributions.

It features the implementation of some of the most common packet loss models, PLC algorithms and metrics:

**Packet Loss Simulation**
- **Binomial**: uniform distribution of packet losses, governed by the Packet Error Ratio (PER) parameter.
- **Metronome**: periodic packet losses governed by the burst period, the burst length, and an offset.
- **Gilbert-Elliot**: bursty distribution of packet losses, governed by the four probabilities associated to its two states (For each state, the probability of packet loss and the probability of transitioning to the other state) [[1](#1)].

**PLC Algorithms**
- **Zeros**: the lost samples are replaced by zeros.
- **Last Packet**: the lost samples are replaced by the last received packet.
- **Low-Cost**: implementation of the algorithm proposed in [[2](#2)].
- **Burg**: Python bindings for the [C++ implementation of the Burg method](https://github.com/matteosacchetto/burg-implementation-experiments).
- **Deep Learning**: implementation of the algorithm proposed in [[3](#3)].
- **Advanced**: allows to apply different PLC algorithms to different frequency bands and audio channels (M/S processing included).
- **External**: Python bindings for C++ to simplify the integration of existing algorithms.

**Metrics**
- **Mean Square Error**: the mean square error between the original and reconstructed signal.
- **Mean Amplitude Error**: the mean amplitude error between the original and reconstructed signal.
- **PEAQ**: the Perceptual Evaluation of Audio Quality (PEAQ) metric, as defined in [[4](#4)].
- **Windowed PEAQ**: PEAQ with a specific window length.
- **Spectral Energy Difference**: Difference of spectral energy between the original and reconstructed signal. (Currenty not usable)
- **Human**: this metric produces the config file for a MUSHRA test using as stimuli excerpts of the reconstructed audio tracks. It also gathers the results of the test to be displayed alongside the other metrics.
- **Perceptual**: perceptually-motivated evaluation metric for Packet Loss Concealment in Networked Music Performances, as defined in [[5](#5)]

## Installation

You will need a mongoDB database to store the results. You can install it locally or use a cloud service like [MongoDB Atlas](https://www.mongodb.com/cloud/atlas).
It is recomended however to use the [Docker image](https://hub.docker.com/_/mongo) provided by MongoDB.

Install WSL (https://learn.microsoft.com/de-de/windows/wsl/install).
```bash
    wsl --install
    wsl.exe -d Ubuntu
```
Choose your username and password and go to the path where you want to save your files. Create an isolated environment with Python 3.10 and activate it.
```bash
    sudo apt install python3.10 python3.10-venv python3.10-dev
    python3.10 -m venv venv310
    source venv310/bin/activate
```

Update package sources, install additional tools, add external Python package source and update again.
```bash
    sudo apt update
    sudo apt install podman
    sudo apt install software-properties-common
    sudo apt install jupyter-core
    sudo apt install libsndfile1
    sudo apt install pipx
    sudo apt install python3-pip
    sudo snap install astral-uv --classic
    sudo add-apt-repository ppa:deadsnakes/ppa
    sudo apt update
```

Install podman and pull the image.
```bash
    podman pull docker.io/library/mongo:6.0.8
```
Then run the container setting the port to 27017 and the name to mongodb. Also set the username and password for the database.
```bash
    podman run -d -p 27017:27017 --name mongodb \
    -e MONGO_INITDB_ROOT_USERNAME=myUserAdmin \
    -e MONGO_INITDB_ROOT_PASSWORD=admin \
    mongo:6.0.8
```

Clone this repository, install the requirements and the plctestbench package.

```bash
    git clone https://github.com/marcel-ellert/plc-testbench
    cd plc-testbench
    pip install -r requirements.txt
    cd ..
```

Clone and install the [burg-python-bindings](https://github.com/LucaVignati/burg-python-bindings).
```bash
    git clone https://github.com/LucaVignati/burg-python-bindings.git
    cd burg-python-bindings
    python setup.py install
    cd ..
```

Clone and install the [cpp_plc_template](https://github.com/LucaVignati/cpp_plc_template).
```bash
    git clone https://github.com/LucaVignati/cpp_plc_template.git
    cd cpp_plc_template
    python setup.py install
    cd ..
```

Install the GSTREAMER library and the PEAQ plugin.
```bash
    sudo apt-get install libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev libgstreamer-plugins-bad1.0-dev gstreamer1.0-plugins-base gstreamer1.0-plugins-good gstreamer1.0-plugins-bad gstreamer1.0-plugins-ugly gstreamer1.0-libav gstreamer1.0-tools gstreamer1.0-x gstreamer1.0-alsa gstreamer1.0-gl gstreamer1.0-gtk3 gstreamer1.0-qt5 gstreamer1.0-pulseaudio git gtk-doc-tools git2cl automake libtool
    mkdir gstpeaq && git clone https://github.com/HSU-ANT/gstpeaq.git gstpeaq
    cd gstpeaq
    aclocal && autoheader && ./autogen.sh && sed -i 's/SUBDIRS = src doc/SUBDIRS = src/' Makefile.am && ./configure --libdir=/usr/lib && automake && make && make install
    echo 'export GST_PLUGIN_PATH=/usr/lib/gstreamer-1.0:$GST_PLUGIN_PATH' >> ~/.bashrc
    source ~/.bashrc
    cd ..
```

Install webMUSHRA and pyMUSHRA.
```bash
    cd plc-testbench
    mkdir listening_tests && cd listening_tests
    mkdir db
    git clone https://github.com/audiolabs/webMUSHRA.git webmushra
    git clone https://github.com/nils-werner/pymushra.git pymushra
    pip install -e pymushra
    cd webmushra
	podman build -t webmushra -f Dockerfile .
    cd ../..
```

## Usage

### Startup and Initializing

The file `plctestbench.ipynb` contains a Jupyter Notebook. Start the Juypter Notebook.
```bash
    jupyter notebook
```
Copy and paste one of the URLs to a browser. Click on 'plctestbench.ipynb'. This file contains examples and explanations of how to use the tool.
Change the path to your root folder.
```python/jupyter notebook
    testbench_settings = {
    'root_folder': 'path/to/root/folder',
    'db_ip': 'ip.of.the.database',
    'db_port': 27017,
    'db_username': 'myUserAdmin',
    'db_password': 'admin',
    }
```
Put the audio files to be analyzed in this folder and list them as follows (path relative to `root_folder`):
```python/jupyter notebook
original_audio_tracks = [(OriginalAudio, OriginalAudioSettings('Blues_Drums.wav')),
                         (OriginalAudio, OriginalAudioSettings('Blues_Piano.wav'))]
```
Afterwards you can start the Testbench with your specific settings by commiting/uncomitting and changing the setup to your specific needs.
You will find both the audio files and the results in the folder specified in the `root_folder` setting.

Important Information: The DeepLearningPLC algorithm requires the `bufer_size` to be set to 128 in the `Settings` of the `PacketLossSimulator` of choice.

### Crossfades

When a packet is lost, the PLC algorithm has to reconstruct the lost samples. However, the reconstructed samples are not going to be identical to the original ones. This is why we implemented a crossfade feature, which allows to gradually transition from the original samples to the reconstructed ones, and vice versa.

The crossfade can be customized in the following aspects:
- **Length**: the duration of the crossfade in milliseconds.
- **Function**: the function to use:
    - **Power**: the crossfade is $x^n$.
    - **Sinusoidal**: the crossfade is sinusoidal ($\sin\left(x\right)$ for $0 < x < \frac{\pi}{2}$).
- **Exponent**: the exponent of the power crossfade function.
- **Type**:
    - **Equal Power**: the sum of the squares of the two crossfades is equal to 1.
    - **Equal Amplitude**: the sum of the two crossfades is equal to 1.

The crossfade can be applied to the left or the right of the lost samples, or both. The following example illustrates how to configure the settings to use the crossfade:
```python
    crossfade_settings = ManualCrossfadeSettings(length=1,\
                        function='power', exponent=2.0, type='power')

    (ZerosPLC, ZerosPLCSettings(fade_in=QuadraticCrossfadeSettings(length=1),\
        crossfade=crossfade_settings))
```
As shown in the previous example, the `fade_in` parameter of any `PLCAlgorithm` determines the crossfade to apply to the left of the lost samples, while the `crossfade` parameter determines the crossfade to apply to the right of the lost samples. It is important to note that the length of the `fade_in` crossfade cannot be greater than the length of the lost packet.

The `ManualCrossfadeSettings` class allows to manually set the parameters of the crossfade.
However, there are other convenience classes that are pre-set to some common configurations:
- `NoCrossfadeSettings`: disables the crossfade.
- `LinearCrossfadeSettings`: applies a linear crossfade (`function` = 'power' and `exponent` = 1.0).
- `QuadraticCrossfadeSettings`: applies a quadratic crossfade (`function` = 'power' and `exponent` = 2.0).
- `CubicCrossfadeSettings`: applies a cubic crossfade (`function` = 'power' and `exponent` = 3.0).
- `SinusoidalCrossfadeSettings`: applies a sinusoidal crossfade (`function` = 'sinusoidal').


Both the `fade_in` and `crossfade` parameters always default to `NoCrossfadeSettings` so that the crossfade is disabled by default.

### Multiband Crossfade

Different crossfade settings can be applied to different frequency bands. This can be useful mitigate some of the artifacts introduced by the crossfade.

This is an example configuration for three bands crossfade:
```python
    multiband_crossfade_settings = \
        [MultibandSettings(frequencies = [200, 2000]),
         QuadraticCrossfadeSettings(length=50),
         QuadraticCrossfadeSettings(length=5),
         QuadraticCrossfadeSettings(length=1)]

    (ZerosPLC, ZerosPLCSettings(crossfade=multiband_crossfade_settings))
```
The `MultibandSettings` class allows to specify the frequencies of the bands. The first frequency is the upper bound of the first band, while the last frequency is the lower bound of the last band. The number of frequencies determines the number of bands.

The list beginning with the `MultibandSettings` class contains the crossfade settings for each band. The first element of the list is the crossfade settings for the first band, the second element is the crossfade settings for the second band, and so on. The length of the list must be equal to the number of bands.

Each band can have its own crossfade settings, totally unrelated to the other bands.

### Advanced PLC

The `AdvancedPLC` object allow for two complex beheviours simultaneously:
- **Multiband PLC**: different PLC algorithms can be applied to different frequency bands. 
- **Spatial PLC**: different PLC algorithms can be applied to different audio channels (featuring M/S processing).

An example configuration of the settings to achieve such beheaviours is the following:
```python
    frequencies = {'mid': [200, 2000], 'side': [1000]}
    band_settings = {\
        'mid':
            [(ZerosPLC, ZerosPLCSettings()),
             (BurgPLC, BurgPLCSettings(order=512)),
             (BurgPLC, BurgPLCSettings(order=256))],
        'side':
            [(LastPacketPLC, LastPacketPLCSettings()),
             (BurgPLC, BurgPLCSettings(order=256))]}

    (AdvancedPLC, AdvancedPLCSettings(band_settings, frequencies = frequencies, channel_link=False, stereo_image_processing = StereoImageType.MID_SIDE))
```
The `frequencies` parameter is a dictionary that maps the name of the channel to a list of frequencies.

The `band_settings` parameter is a dictionary that maps the name of the channel to a list of tuples. Each tuple contains the PLC algorithm to apply to the band and its settings. The length of each list must be equal to the number of bands of that channel.

If L/R or M/S channels require different PLC algorithms, the `channel_link` parameter needs to be set to `False` and the related settings need to be specified in the `band_settings` parameter using the `left` and `right` or `mid` and `side` keys.

Alternatively, if the same PLC algorithms need to be applied to both channels (regardless of the L/R or M/S coding), the `channel_link` parameter needs to be set to `True` and the related settings need to be specified in the `band_settings` parameter using the `linked` key.

## References
    
<a id="1">[1]</a>
Elliott, Edwin O. "Estimates of error rates for codes on burst-noise channels." The Bell System Technical Journal 42.5 (1963): 1977-1997.

<a id="2">[2]</a>
Fink, Marco, and Udo Zölzer. "Low-Delay Error Concealment with Low Computational Overhead for Audio over IP Applications." DAFx. 2014.
    
<a id="3">[3]</a> 
Verma, Prateek, et al. "A deep learning approach for low-latency packet loss concealment of audio signals in networked music performance applications." 2020 27th Conference of open innovations association (FRUCT). IEEE, 2020.
    
<a id="4">[4]</a> 
Thiede, Thilo, et al. "PEAQ-The ITU standard for objective measurement of perceived audio quality." Journal of the Audio Engineering Society 48.1/2 (2000): 3-29.

<a id="5">[5]</a> 
Vignati, Luca, and Luca Turchet, "On the lack of a perceptually-motivated evaluation metric for Packet Loss Concealment in Networked Music Performances." Journal of the Audio Engineering Society, 2025.