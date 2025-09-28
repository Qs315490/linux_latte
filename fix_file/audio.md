# 驱动加载顺序调整
/etc/modules-load.d/modules.conf 新起一行填写
```
snd_soc_rt5659
```
# 声音修复
注意。如果开机后右扬声器没声音，请执行以下命令。无需root，普通用户执行。
```bash
amixer -c0 cset "name='Amp Input1'" Right
```
/usr/share/alsa/ucm2/conf.d/cht-bsw-rt5659/ 文件夹下
cht-bsw-rt5659.conf
```
Syntax 3

SectionUseCase."HiFi" {
	File "HiFi.conf"
	Comment "Default"
}
```
HiFi.conf
```
SectionVerb {
	If.Controls {
		Condition {
			Type ControlExists
			Control "name='media0_in Gain 0 Switch'"
		}
		Before.EnableSequence "0"
		True {
			Include.pe.File "/platforms/bytcr/PlatformEnableSeq.conf"
			Include.pd.File "/platforms/bytcr/PlatformDisableSeq.conf"
		}
	}

    EnableSequence [
		cset "name='codec_out0 Gain 0 Volume' 72%"
		cset "name='IF2 ADC Mux' DAC_REF"
		cset "name='IF3 ADC Mux' IF_ADC1"
		# 音量50%
		# cset "name='DAC1 Playback Volume' 50%"
		# 设置右扬声器使用右声道
		cset "name='Amp Input1' Right" 
    ]

	DisableSequence [

	]

	Value {
		PlaybackPCM "hw:${CardId}"
		# The speaker ampl. path on the 5659 has no speaker vol control
		# Use the digital DAC1 master control as MixerElem
		# PlaybackMixerElem "DAC1"
		# PlaybackVolume "DAC1 Playback Volume"
		# PlaybackSwitch "DAC1 Playback Switch"
		# PlaybackVolumeScale "0-155:0-100%log"
	}
}

SectionDevice."Speaker" {
	Comment "Stereo Speakers"

	ConflictingDevice [
		# 还没实现，取消注释就报错了
		# "Headphones"
	]

	Value {
		# The speaker ampl. path on the 5659 has no speaker vol control
		# Use the digital DAC1 master control as MixerElem
		PlaybackMixerElem "DAC1"
		PlaybackVolume "DAC1 Playback Volume"
		PlaybackSwitch "DAC1 Playback Switch"
	}

	EnableSequence [
		cset "name='DAC1 Playback Switch' on"
	]
	DisableSequence [
		cset "name='DAC1 Playback Switch' off"
	]
}
```
# 实验性设置方法
复制audio下所有文件，保持目录结构复制到/usr/share/alsa/ucm2  
修改文件所有者为root，组root。权限644

# ~~设置声卡模式~~ 以下内容不需要了，已经修复
## 安装声音调整面板
新发行版使用 `PipeWire` 代替 `PulseAudio`。无需安装
```bash
apt install pavucontrol
```
## 将声卡模式设置为专业音频
配置 -> Built-in Audio -> 专业音频(Pro Audio)  
pipewire版本高于 `1.2.7(大概)` 就有 `专业音频(Pro Audio)`

至此声音设备正常识别
