Android拍照音频文件位于\frameworks\base\data\sounds\effects目录，更具不同的平台区分不同音频文件。

例如拍照声音文件位于\frameworks\base\data\sounds\effects\ogg\

编译后的路径system/media/audio/ui/

```
./AudioTv.mk:    $(LOCAL_PATH)/effects/ogg/camera_click_48k.ogg:system/media/audio/ui/camera_click.ogg \
./AudioPackage11.mk:    $(LOCAL_PATH)/effects/material/ogg/camera_click_48k.ogg:system/media/audio/ui/camera_click.ogg \
./AllAudio.mk:    $(LOCAL_PATH)/effects/ogg/camera_click_48k.ogg:system/media/audio/ui/camera_click.ogg \
./AudioPackage10.mk:    $(LOCAL_PATH)/effects/material/ogg/camera_click_48k.ogg:system/media/audio/ui/camera_click.ogg \
```