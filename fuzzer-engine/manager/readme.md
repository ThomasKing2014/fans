# Fuzzer Manager

In the paper, we do not implement the automatic reflash smartphones functionality. To improve the efficiency of the fuzzer, we now can reflash the smartphones automatically to some extent. 

To make it easier, here we also adjust the fuzzer manager to test the smartphone of the `same model`, e.g., Pixel 2 XL with Android 9.0.0_r46. If you want to test other models, e.g., Pixel 3 XL, you need to build them alone and provide the corresponding images. Besides, you should adjust the fuzzer manager correspondingly.

## Required Libraries

Install the following libraries

```bash
# step1. install pwntools according to https://github.com/Gallopsled/pwntools#installation
apt-get update
apt-get install python3 python3-pip python3-dev git libssl-dev libffi-dev build-essential
python3 -m pip install --upgrade pip
python3 -m pip install --upgrade pwntools
# step2. install adb-sync according to https://github.com/google/adb-sync
```

## Prepare Mobile Phone
Please prepare at least a google pixel series mobile phone, e.g., Pixel 2 XL. We have tested FANS on Pixel, Pixel 2 XL, Pixel 3 XL. For other google pixel series, it should work.

## Flash Mobile Phone

You can utilize `flash.py` to flash the mobile phone. Before flashing, you need to create a config file `flash.cfg` to config the following options.
- `factory_image_path`, the path of the factory image. 
- `original_image_path`, the path of the manually build image without ASan enabled.
- `sanitizer_image_path`, the path of the manually build image with ASan enabled.
- `twrp_img_path`, the twrp image corresponding to the targeted mobile phone.
- `lunch_command`, the lunch command, e.g., `lunch 50` for aosp_taimen-userdebug.

Here we provide a template file of `flash.cfg`, i.e., `flash.template.cfg`. You can modify it according to your environment.

If you want to automatically flash all the mobile phones, you need to provide the `device.cfg`. The structure of the `device.cfg` is as follows
- XXXXXXXXXXXXXX/YYYYYYYYYYYYYY is the serial number of the device.
- 1/2 is the id number assigned to the device. You can assign the number with what you want.

```
{
    "XXXXXXXXXXXXXX": 1,
    "YYYYYYYYYYYYYY": 2
}
```

When all of the mobile phones are connected to the host, you can also run `python3 get_device_list.py` to get `device.cfg`.

Finally, you can modify `flash.py` according to your demand to flash all mobile phones. Currently, it is customized for Pixel 2 XL.

## Config Fuzzer Manager

Before running the fuzzer manager, we should create a config file `fuzzer.cfg` to config the following options.
- `host_fuzzer_path`, the path of fuzzer on the host.
- `fuzzer_log_level`, the log level of fuzzer, e.g., debug, info, error. Please use the error level as the other two levels will consume lots of disk space.
- `host_model_dir`, the path of the interface model on the host.
- `host_seed_dir`, the path of the seed on the host.
- `device_work_dir`, remote fuzzer work dir on the device. Keep as default.
- `max_instance_number`, how many fuzzers will run on one mobile phone.
- `max_tombstone_count`, how many tombstones can be generated by the smartphone. Deprecated now.
- `root_log_dir`, where will you like to save logs, e.g., logcat logs, tombstones. We recommend not to put it in the repo of fans as the logs will occupy lots of disk space.
- `adbd_restart_script_path`, the path of the `restart_adbd.sh`. Now we do not use it.
- `restart_device_script_path`, the path of the `restart_device.sh`.

Here we provide a template file of `fuzzer.cfg`, i.e., `fuzzer.template.cfg`. You can modify it according to your environment.

## Run Manager

Run fuzzer manager as follows

```bash
python3 manager.py
```

## Fuzzing Results
All of the fuzzing results are located in `root_log_dir`. The structure of the log directory is like follows

```
XXXX-XX-XX-XX-XX-XX
├── Device Serial                 # the device serial
│   ├── 0                         # how many times has this device been flashed
│   │   ├── anr                   # Application Not Responding logs
│   │   ├── fuzzer_log            # fuzzer logs related with fuzzer itself
│   │   ├── logcat                # logcat logs generated by the smartphone
│   │   └── tombstones            # tombstones generated by the smartphone
│   ├── 1
│   │   ├── anr
│   │   ├── fuzzer_log
│   │   ├── logcat
│   │   └── tombstones
│   ├── 2
│   │   ├── anr
│   │   ├── fuzzer_log
│   │   ├── logcat
│   │   └── tombstones
│   └── device_manager.log        # logs generated by the manager
├── Device Serial
...
```

For the introduction of tombstone and anr, you could refer to
- tombstone
  - https://source.android.com/devices/tech/debug
  - http://saurabhsharma123k.blogspot.com/2017/02/native-crash-tombstone-in-android-aosp.html
- anr
  - https://developer.android.com/topic/performance/vitals/anr