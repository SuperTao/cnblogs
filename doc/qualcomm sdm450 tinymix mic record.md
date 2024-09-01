最近需要使用tinymix配置主MIC和副MIC进行录音，一般副MIC都是拿来降噪用的。记录一下副MIC录音的方法。

没有找到高通的音频通路图，只能通过logcat，查看audio_route，然后找到对应的mixer_paths.xml文件进行查看。
```
1、主MIC:
start recording:
tinymix 'DEC1 MUX' 'ADC1'
tinymix 'MultiMedia1 Mixer TERT_MI2S_TX' 1
tinycap /data/main.wav
stop recording:
tinymix 'DEC1 MUX' 'ZERO'
tinymix 'MultiMedia1 Mixer TERT_MI2S_TX' 0

2、副MIC
start recording:
tinymix 'DEC2 MUX' 'ADC2'
tinymix 'MI2S_TX Channels' 'Two'
tinymix 'ADC2 MUX' 'INP3'
tinymix 'MultiMedia1 Mixer TERT_MI2S_TX' 1
tinycap /data/second.wav
stop recording:
tinymix 'DEC2 MUX' 'ZERO'
tinymix 'MI2S_TX Channels' 'One'
tinymix 'ADC2 MUX' 'ZERO'
tinymix 'MultiMedia1 Mixer TERT_MI2S_TX' 0

3、speaker播放
tinymix "RX3 MIX1 INP1" "RX1"

tinymix "SPK" "Switch"

tinymix "PRI_MI2S_RX Audio Mixer MultiMedia1" 1
           
tinyplay test.wav

```

Tony Liu

2018-3-30