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
JUCE, which stands for "Jules' Utility Class Extensions," is a C++ framework and library designed for developing cross-platform audio applications, as well as applications for multimedia, sound, and music. It was originally created by Julian "Jules" Storer and is widely used in the audio and music software development industry.

JUCE provides a comprehensive set of tools and libraries for audio and graphical application development. Some of its key features and components include:

**Cross-Platform Development:** JUCE allows developers to write code once and deploy it on multiple platforms, including Windows, macOS, Linux, iOS, Android, and more.

**Audio and MIDI Support:** JUCE provides functionality for working with audio, MIDI, and various audio plugin formats. It's commonly used for creating virtual instruments, digital audio workstations (DAWs), audio effects plugins, and more.

**User Interface:** JUCE offers a framework for creating user interfaces (UI) for audio applications. This includes customizable graphical components and a GUI editor tool.

**Audio Signal Processing:** JUCE includes tools for working with digital signal processing (DSP), enabling developers to create audio effects and processors.

**Cross-Platform Compatibility:** It abstracts platform-specific code, making it easier to create applications that run consistently on different operating systems.

Community and Licensing: JUCE has a strong community of developers and is available under a dual licensing model: a Personal/Open Source license and a Commercial license, allowing developers to choose the best option for their project.

JUCE is a popular choice for audio and music software development due to its powerful features and its ability to handle complex tasks such as real-time audio processing, GUI development, and cross-platform deployment. It is commonly used by audio plugin developers, music software companies, and individual musicians and sound engineers.

# Understand JUCE Sources

JUCE provides a large number of tool libraries for audio programming tasks, and has a good modular architecture, we can introduce only some of the components according to our needs, next we briefly introduce some of the core modules in juce.

| Module | Comment |
| --- | --- |
|juce_audio_devices | Provides cross-platform abstraction capabilities for audio and MIDI devices, supporting the encapsulation of audio capabilities of various platforms such as Oboe/OpenSL/Core Audio/Direct Sound/Jack Audio. |
| juce_audio_basics | Provides a package of basic audio capabilities |
| juce_audio_processors | The encapsulation of the audio processing unit, in the audio field, different effects or processing tasks are always combined by different Processor, each Audio Processor represents a relatively single processing task, such as EQ / Reverb / Mixer, etc...|
|juce_data_structures | For DAWs, audio tasks always end up with corresponding states, which are organized together as projects. juce::ValueTree is the most important data structure provided by JUCE.|
|juce_dsp | Packages for different DSP algorithms |
|juce_events | Encapsulation of the event loop for platforms, providing cross-platform message queuing features |

# Offical References 

* [Tutorial: Build an audio player](https://docs.juce.com/master/tutorial_playing_sound_files.html)

* [Tutorial: Build a MIDI synthesiser](https://docs.juce.com/master/tutorial_synth_using_midi_input.html)






