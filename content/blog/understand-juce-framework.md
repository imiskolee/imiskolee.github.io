+++
title = 'Understand Audio Programming & Juce Framework'
date = 2021-04-03T11:23:31+08:00
draft = false
categories = ["JUCE","Audio"]
+++

# What's Audio Programming

The field of audio programming is a subdomain of multimedia, encompassing various tasks such as sound synthesis, audio signal processing, and audio plugin development. Common applications in the audio programming field include:

* Music players
* Audio editors
* Digital audio workstations (DAWs)
* MIDI signal processing

Audio programming is a standard I/O task, and depending on the stage, it can be divided into:

**Input:** This deals with where the audio data comes from. It may involve real-time audio capture from a microphone, pulling audio data from the file system (or a remote server), or generating audio data through sound algorithm synthesizers.

**Processing:** Beyond basic audio playback, valuable audio processing applications typically manipulate audio in some way. This can occur in the time domain, involving tasks like audio cutting, sound transformations (e.g., fade in/fade out), or in the frequency domain, which might include audio editing software, EQ (equalization) effects, and more.

**Output:** After processing audio data, it's necessary to send the data somewhere. This could involve exporting it to disk, streaming it to other devices, or real-time playback on audio devices.

In summary, regardless of the type of audio processing program you want to develop, you will always encounter these three fundamental tasks: input, processing, and output.

# Why Audio Programming Is Difficult

Audio programming is indeed a complex and challenging field. It requires programmers to have not only strong general programming skills but also substantial knowledge in vertical domains such as digital signal processing (DSP) and music theory. Developing a real-time audio processing program like a Digital Audio Workstation (DAW) is particularly demanding.

The real-time nature of audio rendering far exceeds that of video. For video, a rendering frequency of 24Hz is sufficient to produce smooth results. However, in the case of audio, this is far from enough. A typical requirement for audio processing programs is that the latency should not exceed 20ms, and in more professional studio environments, it should not exceed 10ms. This implies that audio rendering frequencies need to reach 50-100Hz to reliably output real-time audio. Within this short time frame, the software needs to perform complex audio preprocessing, DSP algorithms, multi-track audio synthesis, and continuously adapt to user interactions. It is indeed a challenging task.

To meet these complex and stringent requirements, there are some established technical tasks that need to be addressed:

Lock-Free Multi-Threaded Audio Rendering Algorithms: To ensure the stability of audio rendering, specialized audio threads are typically used for real-time rendering. In the case of complex tasks, multi-threaded rendering may also be necessary. Since audio content is constantly changing, efficient and reliable synchronization between rendering threads and the user interface (UI) thread is crucial.

Audio Buffer Reuse: Audio devices typically provide a fixed-size memory area (block size) for applications to store rendered results. During audio processing, audio data needs to be passed to different processing subroutines. Careful handling of audio data copying is essential to minimize real-time memory allocation and data copying.

# What's JUCE




# Understand JUCE Source Tree
