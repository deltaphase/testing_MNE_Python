# testing read_raw_kit using SQD files recorded at Taiwan

Thanks to Christian Brodbeck, Teon Brooks, Yasuhiro Haruta, and developers of MNE-Python. Now the the SQD file recored from the MEG system in the Academia Sinica at Taiwan can be read directly using `read_raw_kit`. This report is a demonstration of my experience in using `read_raw_kit` with SQD data recorded in my lab. I aimed to check information of measured data and trigger onsets shown in mne-python and in MegLaboratory (v2.004A).

## Part 1. importing continuous MEG data

1. import a CON file:

```python
import mne
import matplotlib.pyplot as plt

confile = '/Users/kevinhsu/Documents/D/000_toolbox/mnepython_example/aef_004-Denoise2.con'
raw_ = mne.io.kit.read_raw_kit(input_fname = confile, stim = [192])
```

2. check measured MEG data at an arbitrary time at 5 sensors:

```python
start, stop = raw_.time_as_index([1.999, 2.001])  # 1.999 s to 2.001 s data segment
data, times = raw_[:, start:stop]

chns_ = [54, 56, 59, 133, 136]
data = data[chns_,-1]
print data
[  6.70472031e-13   1.12097592e-12   9.90026818e-13   7.22419302e-13
   4.52229376e-13]
```

The data is shown in Tesla. Below is a snapshot showing the MEG measures at the same time at the same sensors, and the third column show the data in fT. It looks good to me!

3. browse raw MEG data:

```python
raw_.plot(n_channels=5)
```

4. The data is recoded using the auditory evoked filed paradigm. The onset of 200Hz and 1000Hz tones can be found in trigger channel 192 and 193, respectively. Let's count the number of onsets of 200Hz condition:

```python
events = mne.find_events(raw_, stim_channel='STI 014')
print events 

35 events found
Events id: [1]
[[  6434      0      1]
 [  8844      0      1]
 [ 11002      0      1]
 ..., 
 [177009      0      1]
 [182218      0      1]
 [184425      0      1]]
```

Below is a snapshot showing onsets in channel 192 in MegLaboratory:

The value of onests found in mne-python seems to be later than that in MegLaboratory. Maybe it is because that the default setting of `read_raw_kit` is to check the offset of signals in trigger channels. Therefore, I add a parameter, slope, while importing CON file:

```python
raw_ = mne.io.kit.read_raw_kit(input_fname = confile, stim = [192], slope = '+')
events = mne.find_events(raw_, stim_channel='STI 014')
print events 

35 events found
Events id: [1]
[[  6381      0      1]
 [  8791      0      1]
 [ 10949      0      1]
 ..., 
 [176956      0      1]
 [182165      0      1]
 [184372      0      1]]
 ```
 
 Now, onsets of stimuli are the same in both platforms ;)
 
 Let's look closer. At 3.5 second, there is an event onset in trigger channel 193. The measured value in channel 192 and 193 are shown below:
 
 in MegLaboratory
 
 
 in mne-python
 ```python
start, stop = raw_.time_as_index([3.499, 3.501])  # 3.499 s to 3.501 s data segment
data, times = raw_[:, start:stop]

chns_ = [192, 193]
data = data[chns_,-1]
print data

[ 0.02685547  4.99755859]
 ```
 
The infomation is correct, as the developer says that measures in trigger channels are in Voltage. Additionally, `read_raw_kit` imports signals of each channel/sensor, and I like the way it works. It allows us to perform noise reduction in python, and adjust onsets of events uisng channels that are connected with audio devices, response pads, and an optical detector. I have check the measured values in these channels and they are all good!

## Part 2. importing continuous MEG data of old format

The MEG system at Academia Sinica was established in 2005, and there was a major upgrade during 2008-2009. Although the layout of MEG sensors are the same, the version and system ID of MegLaboratory is changed. This might be the reason why `read_raw_kit` can not import old files:

```python
confile = '/Users/kevinhsu/Documents/D/000_toolbox/mnepython_example/civf_001-unpack.con'
raw_ = mne.io.kit.read_raw_kit(input_fname = confile,
                               stim = [192], slope = '+')

Extracting SQD Parameters from /Users/kevinhsu/Documents/D/000_toolbox/mnepython_example/civf_001-unpack.con...
Creating Raw.info structure...
Traceback (most recent call last):

  File "<ipython-input-49-f1215174b171>", line 3, in <module>
    stim = [192], slope = '+')

  File "/Users/kevinhsu/anaconda/lib/python2.7/site-packages/mne/io/kit/kit.py", line 814, in read_raw_kit
    preload=preload, stim_code=stim_code, verbose=verbose)

  File "<string>", line 2, in __init__

  File "/Users/kevinhsu/anaconda/lib/python2.7/site-packages/mne/utils.py", line 726, in verbose
    return function(*args, **kwargs)

  File "/Users/kevinhsu/anaconda/lib/python2.7/site-packages/mne/io/kit/kit.py", line 100, in __init__
    info, kit_info = get_kit_info(input_fname)

  File "/Users/kevinhsu/anaconda/lib/python2.7/site-packages/mne/io/kit/kit.py", line 546, in get_kit_info
    (version, revision))

IOError: SQD file format V2R002 not supported.
```

The latest version of MegLaboratory is v2.004 and it can import old SQD files. I come out with a solution: using the latest version of MegLaboratory to save a copy of the old SQD file. After that, I can use `read_raw_kit` to import the copy version of the old SQD file:

```python
confile = '/Users/kevinhsu/Documents/D/000_toolbox/mnepython_example/civf_001-unpack-Denoise2.con'
raw_ = mne.io.kit.read_raw_kit(input_fname = confile, stim = [192])

Extracting SQD Parameters from /Users/kevinhsu/Documents/D/000_toolbox/mnepython_example/civf_001-unpack-Denoise2.con...
Creating Raw.info structure...
Setting channel info structure...
Creating Info structure...
Current compensation grade : 0
Ready.
```

The old file can be imported now! The measured data and signals of triigger channels are correctly imported, too.


