# 驱动加载顺序调整
/etc/modules-load.d/modules.conf 新起一行填写
```
snd_soc_rt5659
```
# 声音修复
注意。如果开机后右扬声器没声音，请执行以下命令。无需root，普通用户执行。
```bash
amixer -c0 cset "name='Amp Input1' Rihgt"
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
		cset "name='Amp Input1' Rihgt" 
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
# ~~设置声卡模式~~ 以下内容不需要了，已经修复
## 安装声音调整面板
```bash
apt install pavucontrol
```
## 将声卡模式设置为专业音频
配置 -> Built-in Audio -> 专业音频(Pro Audio)

# 至此声音设备正常识别，如果没有识别尝试以下配置
## pipewire设备识别不正常
/usr/share/pipewire/pipewire.conf 249行新增。参照Dummy-Driver配置后添加即可
```
    { factory = spa-node-factory
        args = {
            factory.name           = api.alsa.pcm.sink
            node.name              = "alsa_card.platfrom-cht-bsw-rt5659"
            node.description       = "PCM Device"
            media.class            = "Audio/Device"
            api.alsa.path          = "hw:0"
            api.alsa.period-size   = 1024
            api.alsa.headroom      = 0
            api.alsa.disable-mmap  = false
            api.alsa.disable-batch = false
            audio.format           = "S16LE"
            audio.rate             = 48000
            audio.channels         = 2
            audio.position         = "FL,FR"
        }
    }
```
添加完成大概长这样
```
context.objects = [
    #{ factory = <factory-name>
    #    ( args  = { <key> = <value> ... } )
    #    ( flags = [ ( nofail ) ] )
    #    ( condition = [ { <key> = <value> ... } ... ] )
    #}
    #
    # Creates an object from a PipeWire factory with the given parameters.
    # If nofail is given, errors are ignored (and no object is created).
    # If condition is given, the object is created only when the context properties
    # all match the match rules.
    #
    #{ factory = spa-node-factory   args = { factory.name = videotestsrc node.name = videotestsrc node.description = videotestsrc "Spa:Pod:Object:Param:Props:patternType" = 1 } }
    #{ factory = spa-device-factory args = { factory.name = api.jack.device foo=bar } flags = [ nofail ] }
    #{ factory = spa-device-factory args = { factory.name = api.alsa.enum.udev } }
    #{ factory = spa-node-factory   args = { factory.name = api.alsa.seq.bridge node.name = Internal-MIDI-Bridge } }
    #{ factory = adapter            args = { factory.name = audiotestsrc node.name = my-test node.description = audiotestsrc } }
    #{ factory = spa-node-factory   args = { factory.name = api.vulkan.compute.source node.name = my-compute-source } }

    # A default dummy driver. This handles nodes marked with the "node.always-process"
    # property when no other driver is currently active. JACK clients need this.
    { factory = spa-node-factory
        args = {
            factory.name    = support.node.driver
            node.name       = Dummy-Driver
            node.group      = pipewire.dummy
            priority.driver = 20000
            #clock.id       = monotonic # realtime | tai | monotonic-raw | boottime
            #clock.name     = "clock.system.monotonic"
        }
    }
    { factory = spa-node-factory
        args = {
            factory.name    = support.node.driver
            node.name       = Freewheel-Driver
            priority.driver = 19000
            node.group      = pipewire.freewheel
            node.freewheel  = true
        }
    }

    { factory = spa-node-factory
        args = {
            factory.name           = api.alsa.pcm.sink
            node.name              = "alsa_card.platfrom-cht-bsw-rt5659"
            node.description       = "PCM Device"
            media.class            = "Audio/Device"
            api.alsa.path          = "hw:0"
            api.alsa.period-size   = 1024
            api.alsa.headroom      = 0
            api.alsa.disable-mmap  = false
            api.alsa.disable-batch = false
            audio.format           = "S16LE"
            audio.rate             = 48000
            audio.channels         = 2
            audio.position         = "FL,FR"
        }
    }

    # This creates a new Source node. It will have input ports
    # that you can link, to provide audio for this source.
    #{ factory = adapter
    #    args = {
    #        factory.name     = support.null-audio-sink
    #        node.name        = "my-mic"
    #        node.description = "Microphone"
    #        media.class      = "Audio/Source/Virtual"
    #        audio.position   = "FL,FR"
    #        monitor.passthrough = true
    #    }
    #}

    # This creates a single PCM source device for the given
    # alsa device path hw:0. You can change source to sink
    # to make a sink in the same way.
    # { factory = adapter
    #     args = {
    #         factory.name           = api.alsa.pcm.source
    #         node.name              = "alsa-source"
    #         node.description       = "PCM Source"
    #         media.class            = "Audio/Source"
    #         api.alsa.path          = "hw:0"
    #         api.alsa.period-size   = 1024
    #         api.alsa.headroom      = 0
    #         api.alsa.disable-mmap  = false
    #         api.alsa.disable-batch = false
    #         audio.format           = "S16LE"
    #         audio.rate             = 48000
    #         audio.channels         = 2
    #         audio.position         = "FL,FR"
    #     }
    # }

    # Use the metadata factory to create metadata and some default values.
    #{ factory = metadata
    #    args = {
    #        metadata.name = my-metadata
    #        metadata.values = [
    #            { key = default.audio.sink   value = { name = somesink } }
    #            { key = default.audio.source value = { name = somesource } }
    #        ]
    #    }
    #}
]
```
