---
layout: post
title: "Building onto SoundPy: Separating Speech"
date: 2021-01-19
published: true
--- 

# Baby Babbles!

In this post, I will show how I take this loooooong audio signal of an adorable baby babbling,

<blockquote class="imgur-embed-pub" lang="en" data-id="7GXvYKX"><a href="https://imgur.com/7GXvYKX">View post on imgur.com</a></blockquote><script async src="//s.imgur.com/min/embed.js" charset="utf-8"></script>

remove the silences,

<blockquote class="imgur-embed-pub" lang="en" data-id="eIEBUH5"><a href="https://imgur.com/eIEBUH5">View post on imgur.com</a></blockquote><script async src="//s.imgur.com/min/embed.js" charset="utf-8"></script>

aaaaand separate the utterances into separate, beautiful chunks!

<blockquote class="imgur-embed-pub" lang="en" data-id="O9xAX3F"><a href="https://imgur.com/O9xAX3F">View post on imgur.com</a></blockquote><script async src="//s.imgur.com/min/embed.js" charset="utf-8"></script>

(Oh my God. This baby babble is gooooorgeous.)

# Intro, Installation, Audio

Given my background in language development, I am delighted to apply <a href="https://aislynrose.bitbucket.io/">SoundPy</a> to some speech processing! 

In order to replicate what I've done, you will need to install SoundPy and find an audio file to work with. This one from <a href="https://freesound.org/people/thomaslau/sounds/371958/">freesound</a> should work nicely. Just be sure to clip it. It's a bit long.

Soon, you will be able to install SoundPy via `pip install soundpy==v0.1.0a3`. (If it's much later than January 2021, then go ahead and give that a goo.)

As of January, 2021, I have not released an updated SoundPy :( That means that you will need to install the development version, straight from GitHub. 

Let's start up a virtual environment.

```
$ virtualenv -p python3.8 env
$ source env/bin/activate
(env)...$ pip install git+https://github.com/a-n-rose/Python-Sound-Tool.git@development
```
At the bottom of this post, I have written aaaaaall the functions applied in these snippets of code. Copy and paste that code into a python file named `audio_processing.py` and you should be able to import it as I do below.


# Load and Plot Audio

This example uses a Python file, but you can also run this in ipython. However, there are issues in the latest releases of dependencies. You can fix this with the following:

```
(env)...$ pip install -U jedi==0.17.2 parso==0.7.1 ipython
```

Back to the example. 

In a Python file type up the following. Let's save it as `speech_separation.py`. 

```
import soundpy as sp

sr = 44100 # you can try reducing this but this hasn't been tested
# limit to 1 minute
audio, sr = sp.loadsound(<your_audio_filename>, sr=sr, dur_sec = 60) 
sp.feats.plot(audio, sr=sr, feature_type='signal')
```
If you run this:

```
(env)..$ python3 speech_separation.py
```
you should see a plot somewhat like this one, but resembling your audio file.

<blockquote class="imgur-embed-pub" lang="en" data-id="7GXvYKX"><a href="https://imgur.com/7GXvYKX">View post on imgur.com</a></blockquote><script async src="//s.imgur.com/min/embed.js" charset="utf-8"></script>

# Voice Activity Detection and Silence Removal 

Here is where my new functions get applied:

```
import audio_processing as ap # see code at end of post

audio_VAD, utterance_matrix, sr = ap.separate_utterances(
    audio, sr, extend_window_ms = 350, win_size_ms = 100)
```
I'll plot `audio_VAD`:

```
sp.feats.plot(audio_VAD, sr=sr, feature_type='signal')
```
<blockquote class="imgur-embed-pub" lang="en" data-id="eIEBUH5"><a href="https://imgur.com/eIEBUH5">View post on imgur.com</a></blockquote><script async src="//s.imgur.com/min/embed.js" charset="utf-8"></script>

You can see it is already much shorter than the original audio.

```
print(len(audio)) # number of total samples
print(len(audio_VAD))
```
```
4352670
1907325
```

# Utterance Separation!

Now for the fun part. Let's take a look at some utterances! The separated utterances were saved in the variable `utterance_matrix`.

```
sp.feats.plot(utterance_matrix[21], sr=sr, feature_type='signal')
```

Oooooooooooooooh

<blockquote class="imgur-embed-pub" lang="en" data-id="iAK9cYw"><a href="https://imgur.com/iAK9cYw">View post on imgur.com</a></blockquote><script async src="//s.imgur.com/min/embed.js" charset="utf-8"></script>

```
sp.feats.plot(utterance_matrix[11], sr=sr, feature_type='signal')
```

<blockquote class="imgur-embed-pub" lang="en" data-id="3eNn0nE"><a href="https://imgur.com/3eNn0nE">View post on imgur.com</a></blockquote><script async src="//s.imgur.com/min/embed.js" charset="utf-8"></script>

Aaaaaaaaaaaaaaah

Okay, pictures alone aren't so spectacular. 

# Adjusting the Parameters

Now on to the parameters `extend_window_ms` and `win_size_ms`.

## extend_window_ms (a.k.a. PAD the VAD!)

The silences were removed by detecting where voice activity was present. The voice activity algorithm can sometimes cut off speech so this parameter allows you to pad where VAD (voice activity detection) has been indicated. 

If you have a nice recording without many other speakers, you can set extend_window_ms anywhere between 100 and 400 ms. Just depends on your preferences. If you have messier audio with more speakers, you might be happier with 0 to 100.

I wanted to be sure to capture the beginnings and endings of each utterance, so I found

```
extend_window_ms = 350
```

to be quite successful in this case. 

Here is an example of how an utterance was sectioned with the following padding settings:

### extend_window_ms = 350

<blockquote class="imgur-embed-pub" lang="en" data-id="37szUen"><a href="https://imgur.com/37szUen">View post on imgur.com</a></blockquote><script async src="//s.imgur.com/min/embed.js" charset="utf-8"></script>

### extend_window_ms = 200 

<blockquote class="imgur-embed-pub" lang="en" data-id="IVefAlW"><a href="https://imgur.com/IVefAlW">View post on imgur.com</a></blockquote><script async src="//s.imgur.com/min/embed.js" charset="utf-8"></script>

### extend_window_ms = 100

<blockquote class="imgur-embed-pub" lang="en" data-id="VovYQpK"><a href="https://imgur.com/VovYQpK">View post on imgur.com</a></blockquote><script async src="//s.imgur.com/min/embed.js" charset="utf-8"></script>

You can see the end of the utterance got chopped off when set to 100. In some cases, that's okay, for example if otherwise you will catch an additional speaker. When set to 350, the utterance contains two baby babbles. That's fine for me, as long as it's the same baby, as this was clearly a full sentence.


## win_size_ms --> Increase / Decrease VAD sensitivity

The audio samples are processed in little windows. For speech recognition, where you want to identify different speech sounds in the signal, a small window is necessary to catch the quick changes, e.g. 16-25 ms. 

Buuut, if the fast changes aren't important, larger windows are useful, for example, when wanting to know if silence is there or speech. Tinkering around I have found 

```
win_size_ms = 100
```
to work nicely. 

This is more relevant for how sensitive the voice activity detector is. If you want the VAD to be more sensitive to speech, set this value lower, for example 50 - 100. If you want the VAD to be more sensitive to silences, set this value higher, for example, 300. 

You can see the difference in the number of utterances detected in the following snippets:

```
audio_VAD, utterance_matrix, sr = ap.separate_utterances(
    audio, sr, extend_window_ms = 0, win_size_ms = 100)
print(utterance_matrix.shape)
```

```
(60, 50715)
```
A total of 60 detected utterances and...

```
audio_VAD, utterance_matrix, sr = ap.separate_utterances(
    audio, sr, extend_window_ms = 0, win_size_ms = 300)
print(utterance_matrix.shape)
```
```
(36, 66150)
```
A total of 36 detected utterances.

For reference, the settings I chose resulted in the following:

```
audio_VAD, utterance_matrix, sr = ap.separate_utterances(
    audio, sr, extend_window_ms = 350, win_size_ms = 100)
print(utterance_matrix.shape)
```
```
(26, 154350)
```
totalling in 26 utterances. Keep in mind, the detected VAD sections were padded with 350 ms, which ended up combining originally separated utterances.

It just depends on your purpose and taste, really.

# Sum Up

I hope you have fun tinkering around with - and perhaps improving - this code. Please forgive any bugs that pop up. 

In the future I will look into individual utterance analysis. I am sure I'll find my baby's first word in there somewhere...

Happy sound chunking!

# Code for `audio_processing.py`

This isn't perfect code, but it's good enough. Muh buhuh's hungry.

```
import soundpy as sp
import numpy as np


def separate_utterances(audio, sr, vad_stft = False,
                        percent_overlap = 0.5, win_size_ms = 100,
                        extend_window_ms = 0):
    '''Use silences to separate vocal sounds.
    
    For more consistent VAD performance, improved performance has been 
    found with win_size_ms to100 ms and percent_overlap set to 0.5. 
    
    Parameters
    ----------
    audio : numpy array
        Audio samples 
        
    sr : int or float
        Sample rate of audio samples.
        
    win_size_ms : int, float
        The length in milliseconds each processing window should be for VAD.
        In sum, VAD increases in sensitivity with a larger win_size_ms (e.g.
        100) and decreases in sensitivity with smaller win_size_ms (e.g. 50). 
        
    percent_overlap : float
        The percent each processing window should overlap for VAD. Options:
        0.5 or 0. 
        
    extend_window_ms : int, float
        The amount in milliseconds the VAD should be padded. This may be desirable 
        if the VAD cuts speech off.
    
    Returns
    -------
    data_vad : numpy.ndarray
        Data where voice activity was detected - silences removed.
    
    vad_sectioned : numpy.ndarray
        The separate utterances stored within a numpy matrix, shape
        (num utterances, length of longest utterance)
        
    sr : int 
        The sample rate of the audio samples.
    '''
    if not isinstance(audio, np.ndarray):
        raise TypeError('Provided `audio` must be a numpy array. '+\
            'Received input is type {}.'.format(type(audio)))
    if not isinstance(sr, int):
        raise TypeError('Provided `sr` must be an integer. '+\
            'Received input is type {}.'.format(type(sr)))
    # this takes a while. TODO: speed VAD up.
    vad_matrix, vad_settings = sp.dsp.vad(audio, sr, 
                            win_size_ms = win_size_ms, percent_overlap = percent_overlap)
    if vad_stft:
        data_vad, vad_sectioned = utterance_matrix_stft(
            audio, sr=sr, win_size_ms = win_size_ms,
            percent_overlap = percent_overlap, extend_window_ms = extend_window_ms)
    else:
        data_vad, vad_sectioned = utterance_matrix_samples(
            audio, vad_matrix, sr=sr, win_size_ms = win_size_ms,
            percent_overlap = percent_overlap, extend_window_ms = extend_window_ms)
    return data_vad, vad_sectioned, sr

def separate_arrays(array):
    '''Takes array of indices and separates where consecutive indices stop
    
    Parameters
    ----------
    array : numpy.ndarray or array like
        A list of ascending numbers with breaks in between.
        
    Returns
    -------
    array_arrays : list
        Python list containing separate lists, each of which only
        contain consecutive numbers.
    
    Example
    -------
    >>> array = [0, 1, 2, 3, 4, 6, 7, 8, 10, 11, 34, 35, 36]
    >>> separate_arrays(array)
    [[0, 1, 2, 3, 4], [6, 7, 8], [10, 11]]
    '''
    array_arrays = []
    index_prev = 0
    array_curr = []
    for i, index in enumerate(array):
        if index == 0 or i == 0:
            array_curr.append(index)
            index_prev = index
        elif index_prev == index - 1:
            array_curr.append(index)
            index_prev = index
            if index == len(array)-1:
                # array_curr is full --> append and set to empty
                array_arrays.append(array_curr)
                array_curr = []
        else:
            # indices are not consecutive
            # store old array_curr and start new array_curr
            array_arrays.append(array_curr)
            array_curr = [index]
            index_prev = index
    return array_arrays

def arrays2matrix(array_arrays):
    '''creates numpy matrix based on longest array; zerpads other arrays.
    
    Parameters
    ----------
    array_arrays : list, array - like
        A set of lists, e.g. of numbers, to be transferred to numpy matrix.
        
    Returns
    -------
    np_matrix : numpy.ndarray
        Numpy matrix, shape (length of array_arrays, length longest array)
        
    Example
    -------
    >>> array_arrays = [[0, 1, 2, 3, 4], [6, 7, 8], [10, 11]]
    >>> arrays2matrix(array_arrays)
    [[ 0.  1.  2.  3.  4.]
    [ 6.  7.  8.  0.  0.]
    [10. 11.  0.  0.  0.]]
    '''
    max_len = 0
    for i, array in enumerate(array_arrays):
        if len(array) > max_len:
            max_len = len(array)
            
    np_matrix = np.zeros((len(array_arrays),max_len))
    for row in range(len(array_arrays)):
        np_matrix[row,:len(array_arrays[row])] = array_arrays[row]

    return np_matrix


def index_extract(array_orig, array_indices):
    '''From array or arrays of indices, select sections from original array.
    
    If array_indices is zeropadded, the zeros must be padded on the right.
    
    Parameters
    ----------
    array_orig : numpy.ndarray
        The original array to extract data from.
        
    array_indices : numpy.ndarray 
        The set of arrays containing indices where target data is located.
        
    Returns
    -------
    sections : numpy.ndarray
        The data from `array_orig` in the same shape as `array_indices`.
        
    Example
    -------
    >>> orig_samps = np.array([10,11,12,13,14,15])
    >>> sectioned_indices = np.array([[0,0,0],[1,0,0],[2,3,0],[4,5,0]])
    >>> index_extract(orig_samps, sectioned_indices)
    [[10 0 0]
    [11 0 0]
    [12 13 0]
    [14 15 0]]
    '''
    # can be int, float, or complex
    data_type = array_orig.dtype
    if len(array_orig.shape) > 1 and len(array_indices.shape) > 1:
        sectioned_shape = (array_indices.shape[0], # num utterances
                           array_indices.shape[1], # length of utterance
                           array_orig.shape[1]) # stft features of the utterance
    else:
        sectioned_shape = array_indices.shape
    # assuming array_indices is 1D and array_orig is potentially 2D
    sections = np.zeros(sectioned_shape, dtype = data_type)
    for i, row in enumerate(array_indices):
        # expects each row to have ascending indices
        # beyond this max index are zeros
        max_index = np.argmax(row)
        if max_index != len(row) - 1:
            # collect values including max index
            indices = row.astype(int)[:max_index+1]
            sections[i][:max_index+1] += array_orig[indices]
        else:
            # no zeros in row, collect until end of row
            indices = row.astype(int)[::]
            sections[i][::] += array_orig[indices]
    return sections

# TODO improve dealing with windows, overlapping, etc.
# BUG when percent_overlap is set to 0 or win_size_ms set to 50 or 75
def utterance_matrix_samples(original_samples, vad_matrix, sr, 
                      win_size_ms=50, percent_overlap=0.5, extend_window_ms=0, zeropad=True,
                      window = 'Hann'):
    '''Collects samples and separates utterances where voice activity is detected. 
    
    Useful for time domain analysis.
    
    Parameters
    ----------
    original_samples : numpy.ndarray 
        The original samples where VAD has been analyzed
        
    vad_matrix : numpy.ndarray
        The VAD matrix, in frequency domain, of the `original_samples`
        
    sr : int 
        The sample rate of `original_samples`
        
    win_size_ms : int, float 
        The window size that was applied during VAD analysis.
        
    percent_overlap : float 
        The percent the windows should overlap, which was applied during VAD analysis.
        
    extend_window_ms : int 
        The amount VAD detection should be padded, before and after voice activity 
        is detected. This should be the same value applied when calculating the 
        voice activity.
    
    zeropad : bool 
        If True, incomplete rows will be zeropadded. If False, incomplete
        rows will not be included.
        
    window : string
        The window applied when VAD was calculated. (e.g. 'Hann' or 'Hamming')
        
    Returns
    -------
    vad_samples : numpy.ndarray
        The numpy array of the original samples, with silence removed, 
        shape (total VAD samples)
        
    utterance_matrix_samples : numpy.ndarray
        Numpy array with utterances separated into rows and zeropadded,
        shape (number utterances, highest number of samples per utterance)
    '''
    frame_length = sp.dsp.calc_frame_length(win_size_ms, sr)
    num_overlap_samples = int(frame_length * percent_overlap)
    num_subframes = sp.dsp.calc_num_subframes(len(original_samples),
                                                frame_length = frame_length,
                                                overlap_samples = num_overlap_samples,
                                                zeropad = zeropad)
    # set number of subframes for extending window
    extwin_num_samples = sp.dsp.calc_frame_length(extend_window_ms, sr)
    num_win_subframes = sp.dsp.calc_num_subframes(extwin_num_samples,
                                                    frame_length = frame_length,
                                                    overlap_samples = num_overlap_samples,
                                                    zeropad = zeropad)
    samples_matrix = sp.dsp.create_empty_matrix((len(original_samples)),
                                                complex_vals = False)
    
    vad_matrix_extwin = vad_matrix.copy()
    # extend VAD windows with if VAD found
    if extend_window_ms > 0:
        for i, row in enumerate(vad_matrix):
            if row > 0:
                # label samples before VAD as VAD
                if i > num_win_subframes:
                    vad_matrix_extwin[i-num_win_subframes:i] = 1
                else:
                    vad_matrix_extwin[:i] = 1
                # label samples before VAD as VAD
                if i + num_win_subframes < len(vad_matrix):
                    vad_matrix_extwin[i:num_win_subframes+i] = 1
                else:
                    vad_matrix_extwin[i:] = 1
                    
    section_start = 0
    extra_rows = 0
    row = 0
    window_frame = sp.dsp.create_window(window, frame_length)
    vad_sample_indices = np.zeros((len(original_samples)))
    for frame in range(num_subframes):
        vad = vad_matrix_extwin[frame]
        if vad > 0:
            section = original_samples[section_start : section_start + frame_length]
            if percent_overlap > 0:
                # apply overlap add to signal
                section_windowed = sp.dsp.apply_window(section, window_frame, zeropad = zeropad)
                section = section_windowed
                
            # TODO improve issues with mismatching dimensions. Causes the following bug. 
            # BUG when percent_overlap set to 0:
            # ERROR
            # samples_matrix[row : row + frame_length] += section
            # ValueError: operands could not be broadcast together with shapes (4410,) (882,) (4410,) 
            # BUG when win_size_ms set to 50:
            #/home/airos/Projects/gitlab/family-language-tracker/audio_processing/audio_processing.py:297: UserWarning: 

            #Warning: Dimensions don't match. Skipping section row: index 5083727 to index 5085932.

            #warnings.warn("\n\nWarning: Dimensions don't match. Skipping section row: "+\
            #samples matrix shape:  (0,)
            #section to add shape:  (2205,)

            if samples_matrix[row : row + frame_length].shape >= \
                section.shape:
                samples_matrix[row : row + frame_length] += section
                vad_sample_indices[row : row + frame_length] = range(section_start,
                                                                 section_start + frame_length)
            else:
                import warnings
                warnings.warn("\n\nWarning: Dimensions don't match. Skipping section row: "+\
                    "index {} to index {}.\n".format(row, row+frame_length))
                print('samples matrix shape: ',samples_matrix[row : row + frame_length].shape)
                print('section to add shape: ', section.shape)
                print()
            row += (frame_length - num_overlap_samples)
        else:
            extra_rows += frame_length - num_overlap_samples
        section_start += (frame_length - num_overlap_samples)
    if extra_rows == 0:
        print('\nNo silences detected. Try the following: \n'+\
            '* Clean or Filtered Audio: decrease the VAD sensitivity by increasing `win_size_ms`.\n'+\
                '* Noisy Audio: filter background noise using soundpy.filtersignal().\n')
        extra_rows = -len(samples_matrix)
    vad_samples = samples_matrix[:-extra_rows]
    max_index = np.argmax(vad_sample_indices)
    # remove zero values beyond max index
    vad_sample_indices = vad_sample_indices[:max_index+1] 
    
    vad_samples_separated = separate_arrays(vad_sample_indices)
    index_matrix = arrays2matrix(vad_samples_separated)
    utterance_matrix_samples = index_extract(original_samples, index_matrix)
    return vad_samples, utterance_matrix_samples


def utterance_matrix_stft(audio, sr, 
                      win_size_ms=50, percent_overlap=0.5, extend_window_ms=0, zeropad=True,
                      window = 'Hann'):
    '''Collects stft and separates utterances where voice activity is detected. 
    
    Useful for frequency domain analysis.
    
    Parameters
    ----------
    original_samples : numpy.ndarray 
        The original samples where VAD has been analyzed
        
    sr : int 
        The sample rate of `original_samples`
        
    win_size_ms : int, float 
        The window size that was applied during VAD analysis.
        
    percent_overlap : float 
        The percent the windows should overlap, which was applied during VAD analysis.
        
    extend_window_ms : int 
        The amount VAD detection should be padded, before and after voice activity 
        is detected. This should be the same value applied when calculating the 
        voice activity.
    
    zeropad : bool 
        If True, incomplete rows will be zeropadded. If False, incomplete
        rows will not be included.
        
    window : string
        The window applied when VAD was calculated. (e.g. 'Hann' or 'Hamming')
        
    Returns
    -------
    stft_vad : numpy.ndarray
        The numpy array of the original samples, with silence removed, 
        shape (total VAD samples)
        
    utterance_matrix_stft : numpy.ndarray
        Numpy array with utterances separated into rows and zeropadded,
        shape (number utterances, highest number of samples per utterance)
    '''
    stft_vad, vad_matrix = get_VAD_stft(
        audio, sr=sr, vad = True, extend_window_ms = extend_window_ms, 
        win_size_ms = win_size_ms, percent_overlap = percent_overlap)
    
    stft_full = sp.feats.get_stft(audio, sr=sr,
                                  win_size_ms = win_size_ms, 
                                  percent_overlap = percent_overlap)
    
    # should have same number of rows
    assert stft_full.shape[0] == vad_matrix.shape[0]
    
    vad_matrix_indices = np.where(vad_matrix==1)[0]
    vad_indices_separated = separate_arrays(vad_matrix_indices)
    
    utterance_indice_matrix = arrays2matrix(vad_indices_separated)
    utterance_matrix_stft = index_extract(stft_full, utterance_indice_matrix)
    return stft_vad, utterance_matrix_stft

def get_VAD_stft(sound, sr=48000, vad=True, win_size_ms=50, percent_overlap=0.5, real_signal=False, 
                        fft_bins=1024, window='hann', use_beg_ms=120, extend_window_ms=0, energy_thresh=40, 
                        freq_thresh=185, sfm_thresh=5, zeropad=True, **kwargs):
    '''Returns STFT matrix and VAD matrix. STFT matrix contains only non-VAD sections.
    
    Parameters
    ----------
    sound : str or numpy.ndarray [size=(num_samples,) or (num_samples, num_channels)]
        If str, wavfile (must be compatible with scipy.io.wavfile). Otherwise 
        the samples of the sound data. Note: in the latter case, `sr`
        must be declared.
    
    sr : int, optional
        The sample rate of the sound data or the desired sample rate of
        the wavfile to be loaded. (default None)
    
    vad : bool
        If True, STFT with only speech detected windows will be returned. If False, 
        STFT with only background sounds will be returned.
    
    win_size_ms : int or float
        Window length in milliseconds for Fourier transform to be applied
        (default 50)
    
    percent_overlap : int or float 
        Amount of overlap between processing windows. For example, if `percent_overlap`
        is set at 0.5, the overlap will be half that of `win_size_ms`. (default 0.5) 
        If an integer is provided, it will be converted to a float between 0 and 1. 
        
    real_signal : bool 
        If True, only half the FFT spectrum will be used; there should really be no difference
        as the FFT is symmetrical. If anything, setting `real_signal` to True may speed up 
        functionality / make functions more efficient.

    fft_bins : int 
        Number of frequency bins to use when applying fast Fourier Transform. (default 1024)
        
    window : str 
        The window function to apply to each window segment. Options are 'hann' and 'hamming'.
        (default 'hann')
        
    use_beg_ms : int 
        The amount of time in milliseconds to use from beginning of signal to estimate background
        noise.
        
    extend_window_ms : int 
        The amount of time in milliseconds to pad or extend the identified VAD segments. This 
        may be useful to include more speech / sound, if desired.
        
    energy_thresh : int 
        The threshold to set for measuring energy for VAD in the signal. (default 40)
        
    freq_thresh : int 
        The threshold to set for measuring frequency for VAD in the signal. (default 185)
        
    sfm_thresh : int 
        The threshold to set for measuring spectral flatness for VAD in the signal. (default 5)
        
    zeropad : bool 
        If True, samples will be zeropadded to fill any partially filled window. If False, the 
        samples constituting the partially filled window will be cut off.
    
    **kwargs : additional keyword arguments
        Keyword arguments for `soundpy.files.loadsound`
        
    Returns
    -------
    stft_matrix : np.ndarray [size=(num_frames_vad, fft_bins//2+1), dtype=np.complex_]
        The STFT matrix frames of where either VAD or no VAD has been detected.
        
    vad_matrix_extwin : np.ndarray [size=(num_frames,)]
        A vector containing indices of the full STFT matrix for frames of where voice activity 
        was detected or not.
    '''
    # raise ValueError if percent_overlap is not supported
    if percent_overlap != 0 and percent_overlap < 0.5:
        raise ValueError('For this VAD function, `percent_overlap` ' +\
            'set to {} is not currently supported.\n'.format(percent_overlap) +\
                'Suggested to set at either 0 or 0.5')
    if percent_overlap > 0.5:
        import warnings
        msg = '\nWarning: for this VAD function, parameter `percent_overlap` has most success '+\
            'when set at 0 or 0.5'
    # raise warnings if sample rate lower than 44100 Hz
    if sr < 44100:
        import warnings
        msg = '\nWarning: voice-activity-detection works best with sample '+\
            'rates above 44100 Hz. Current `sr` set at {}.'.format(sr)
        warnings.warn(msg)
    if isinstance(sound, np.ndarray):
        data = sound.copy()
    else:
        data, sr2 = sp.loadsound(sound, sr=sr, **kwargs)
        assert sr2 == sr
    frame_length = sp.dsp.calc_frame_length(win_size_ms, sr)
    num_overlap_samples = int(frame_length * percent_overlap)
    num_subframes = sp.dsp.calc_num_subframes(len(data),
                                                frame_length = frame_length,
                                                overlap_samples = num_overlap_samples,
                                                zeropad = zeropad)
    
    # set number of subframes for extending window
    extwin_num_samples = sp.dsp.calc_frame_length(extend_window_ms, sr)
    num_win_subframes = sp.dsp.calc_num_subframes(extwin_num_samples,
                                                    frame_length = frame_length,
                                                    overlap_samples = num_overlap_samples,
                                                    zeropad = zeropad)
    
    total_rows = fft_bins
    if len(data.shape) > 1 and data.shape[1] > 1:
        stereo = True
        stft_matrix = sp.dsp.create_empty_matrix(
            (num_subframes, total_rows, data.shape[1]),
            complex_vals = True)
        # stereo sound --> average out channels for measuring energy
        data_vad = sp.dsp.average_channels(data)
    else:
        stereo = False
        stft_matrix = sp.dsp.create_empty_matrix(
            (num_subframes, total_rows),
            complex_vals = True)
        data_vad = data
    
    

    vad_matrix, (sr, e, f, sfm) = sp.dsp.vad(data_vad, sr, 
                                               win_size_ms = win_size_ms,
                                               percent_overlap = percent_overlap, 
                                               use_beg_ms = use_beg_ms,
                                               energy_thresh = energy_thresh, 
                                               freq_thresh = freq_thresh, 
                                               sfm_thresh = sfm_thresh)
    vad_matrix_extwin = vad_matrix.copy()
    
    # extend VAD windows where VAD indicated
    if extend_window_ms > 0:
        for i, row in enumerate(vad_matrix):
            if row > 0:
                # label samples before VAD as VAD
                if i > num_win_subframes:
                    vad_matrix_extwin[i-num_win_subframes:i] = 1
                else:
                    vad_matrix_extwin[:i] = 1
                # label samples before VAD as VAD
                if i + num_win_subframes < len(vad_matrix):
                    vad_matrix_extwin[i:num_win_subframes+i] = 1
                else:
                    vad_matrix_extwin[i:] = 1
    
    section_start = 0
    extra_rows = 0
    window_frame = sp.dsp.create_window(window, frame_length)
    row = 0
    for frame in range(num_subframes):
        v = vad_matrix_extwin[frame]
        # if vad is True, only detected speech will be included
        # if vad is False, only background sound / noise will be included
        if v == vad: 
            section = data[section_start:section_start+frame_length]
            section = sp.dsp.apply_window(section, 
                                            window_frame, 
                                            zeropad = zeropad)
            section_fft = sp.dsp.calc_fft(section, 
                                            real_signal = real_signal,
                                            fft_bins = total_rows,
                                            )
            stft_matrix[row] = section_fft
            row += 1
        else:
            extra_rows += 1
        section_start += (frame_length - num_overlap_samples)
    stft_matrix = stft_matrix[:-extra_rows]
    return stft_matrix[:,:fft_bins//2+1], vad_matrix_extwin


def calc_snr_signal(audio, sr, win_size_ms = 100, percent_overlap = 0.5, extend_window_ms=300):
    '''Calculate approximate signal to noise ratio in a speech signal.
    
    Note: for higher SNR signals, more accurate results with extend_window_ms around 300;
    for lower SNR signals, extend_window_ms around 200, possibly lower.
    
    Parameters
    ----------
    audio : numpy.ndarray
        Audio samples 
        
    sr : int 
        Sample rate of `audio`.
        
    win_size_ms : int, float
        The size of processing window to apply VAD. Increased sensitivity of VAD 
        is associated with shorter windows (e.g. 25-50). Decreased sensitivity of 
        VAD is associated with longer windows (e.g. 100-300).
    
    percent_overlap : float
        The amount of overlap between processing windows. Options: 0 or 0.5.
    
    extend_window_ms : int
        The amount in milliseconds detected VAD should be padded. 
    
    Warning
    -------
    no background noise detected
        Returns None if no background noise detected
        
    no voice activity detected
        Returns None if no voice activity detected
        
    signal to noise ratio calculation failed
        Returns None if both no background noise or voice activity detected
    '''
    stft_background, vad_matrix = get_VAD_stft(
        audio, sr=sr, vad = False, extend_window_ms = extend_window_ms, 
        win_size_ms = win_size_ms, percent_overlap = percent_overlap)
    stft_vad, vad_matrix = get_VAD_stft(
        audio, sr=sr, vad = True, extend_window_ms = extend_window_ms, 
        win_size_ms = win_size_ms, percent_overlap = percent_overlap)
    if stft_background.shape[0] == 0 or stft_vad.shape[0] == 0:
        import warnings
        if stft_background.shape[0] == 0 and stft_vad.shape[0] > 0:
            warnings.warn('\n\nWarning: no background noise detected. Tips: \n'+\
                '* Increase the setting `win_size_ms` to between 100 and 300.\n'+\
                '* Decrease the setting `extend_window_ms` or `pad_vad_ms` to '+\
                        'between 0 and 100.\n'+\
                '* If audio is very noisy, try filtering the signal, e.g. soundpy.filtersignal.\n'
                '* If audio has been filtered, try decreasing or increasing the filter.\n'+\
                '* If audio has been filtered, try applying postfilter as well.\n'+\
                '* Ensure beginning of audio does not immediately start with speech.\n\n')

        elif stft_vad.shape[0] == 0 and stft_background.shape[0] > 0:
            warnings.warn('\n\nWarning: no voice activity detected. Tips: \n'+\
                '* Decrease the setting `win_size_ms` to between 25 and 100.\n'+\
                '* Increase the setting `extend_window_ms` or `pad_vad_ms` to '+\
                                    'between 100 and 500.\n'+\
                '* If audio is very noisy, try filtering the signal, e.g. soundpy.filtersignal.\n'
                '* If audio has been filtered, try decreasing or increasing the filter.\n'+\
                '* If audio has been filtered, try applying postfilter as well.\n'+\
                '* Ensure beginning of audio does not immediately start with speech.\n\n')
        else:
            warnings.warn('\n\nWarning: signal to noise ratio calculation failed. Tips: \n'+\
                '* Adjust the setting `win_size_ms` to between 25 and 300.\n'+\
                '* Adjust the setting `extend_window_ms` or `pad_vad_ms` to '+\
                                    'between 0 and 500.\n'+\
                '* If audio is very noisy, try filtering the signal, e.g. soundpy.filtersignal.\n'
                '* If audio has been filtered, try decreasing or increasing the filter.\n'+\
                '* If audio has been filtered, try applying postfilter as well.\n'+\
                '* Ensure beginning of audio does not immediately start with speech.\n\n')
        return None
    noise_power = np.abs(stft_background)**2
    target_power = np.abs(stft_vad)**2
    snr = 10 * np.log10(np.mean(target_power)/ (np.mean(noise_power) + 1e-6))
    return np.mean(snr)

```
