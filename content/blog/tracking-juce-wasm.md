+++
title = 'Tracking Juce Wasm'
date = 2023-08-02T10:32:12+08:00
draft = false
categories = ["C++","Audio","JUCE"]
+++

# Why WASM For Audio Programming
正如大家所知道的那样，Web在软件分发方式上具有极大的优势，在现在这个时候，使用web来构建专业的音频应用也已经具备了基础的条件。
* Web Audio API 中已经提供了专门用于开发低延迟的音频应用处理的AudioWorklet API. AudioWorklet工作在独立的线程中，对于保证实时音频处理性能提供了基础的条件。
* WASM。 借助WASM可以直接将原生代码(C++/Rust等)编译为在Web中可以运行的代码，这也就意味着我们的软件可以共享原生平台与Web平台，而无需从头重新开发。这使得我们的原生软件可以实验性地支持Web平台或削弱功能。

基于以上的原因，我尝试将JUCE Framework迁移到了Web平台，并且取得了实际的效果。

# Starting Support WASM On JUCE.
在JUCE中，原生音频设备驱动都封装在`juce_audio_devices::AudioIODevice` 中，这里处理了全部有关音频设备交互的全部功能，包括输入与输出。 对于不同的音频设备类型，则定义在 `juce_audio_devices::AudioDeviceType`中，因此这一部分的支持，我们主要是要：
* 定义一个新的`juce_audio_devicesz::AudioDeviceType`。
* 定义一个新的 `juce_audio_devices::AudioIODevice` 用于port AudioWorklet API。

只要我们完成了以上工作，那么也就代表我们已经完成了而最基本的迁移。

## Support Sound Devices.

## Support Local IO.

# Example Of WASM JUCE


# References

* [AudioWorklet](https://developer.mozilla.org/en-US/docs/Web/API/AudioWorklet)