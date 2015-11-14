# MFFA - Media Fuzzing Framework for Android (Stagefright fuzzer)

## Project overview

The main idea behind this project  is  to create corrupt  but structurally valid media files, direct them to the appropriate software components in Android to  be  decoded  and/or  played  and  monitor  the  system  for  potential  issues  (i.e  system crashes) that may lead to exploitable vulnerabilities. Custom developed Python scripts are used to send the malformed data across a distributed  infrastructure  of  Android  devices,  log  the  findings  and  monitor  for  possible issues, in an automated manner. The actual decoding of the media files on the Android devices is done using the Stagefright command line interface. The results  are sorted out, in an attempt to find only the unique issues, using a custom built triage mechanism.

### System and device configuration 
配置系统和设备

The tool has been developed to be used inside a Linux environment.
这个工具被用在 linux环境下
For the device(s) under test the main problem is including the stagefright command line tool in the Android image that will be flashed on the device(s) or simply building the stagefright module and pushing it to the device.
 测试机的核心问题是， stagefright 命令行工具要被刷到 IMG 镜像中，或者构建一个 stagefright 模块并放到手机中。
There are three alternatives for achieving this goal:
下面有3个可选方案：
 1. if you are building an Android engineering image, you can directly modify the Android.mk file corresponding to the stagefright module. For that you need to go to frameworks/av/cmds/stagefright/ and edit the Android.mk file by looking for the LOCAL_MODULE:=stagefright entry and modifying its corresponding LOCAL_MODULE_TAGS entry from optional to eng. Note that this will NOT work if you are trying to build an user or userdebug Android image.
```
    #LOCAL_MODULE_TAGS := optional
    LOCAL_MODULE_TAGS := eng
    LOCAL_MODULE:= stagefright
```
1.如果你正在编译 android工程镜像，你可以直接修改stagefright模块对应的Android.mk文件，你应该进入 frameworks/av/cmds/stagefright/目录下然后修改 Android.mk 文件中 LOCAL_MODULE:=stagefright 块对应的 LOCAL_MODULE_TAGS标签，从 optional 改成 eng(工业).注意如果你正试图去编译一个 user 或者 userdebug 版本的 android 镜像，这个模块将不起作用。
 2. the second alternative is to go to device/<vendor>/<target_product> and modify the device.mk file by adding the stagefright module to the PRODUCT_PACKAGES entry
```
    PRODUCT_PACKAGES += \
        stagefright
```
2.第二种替代方法就是去修改/device/<vendor>/<target_product>目录下的 device.mk 文件，在 stagefright模块添加PRODUCT_PACKAGES。
 3. Another alternative, and the most straight-forward that doesn't require building the entire image would be to go to the stagefright directory inside the Android tree (frameworks/av/cmds/stagefright) and build only the stagefright module using:
```
	mma (or alternatively)
	mma -B
```
3. 另一种替代方法，大部分简单易懂不需要编译整个镜像，直接进入android目录树的stagefright目录并且只编译stagefright模块使用上面的命令。
### Tool configuration
### 工具配置
Before starting the actual fuzzing campaign there are several configuration files that need to be taken care of:
在开始真正的fuzzing战役前，有几个配置文件需要注意：
1. Firstly, you need to manually run adb devices > devices.txt to the devices.txt config file with the ids of the Android devices that will be used during testing
1. 首先，你需要手动运行 adb devices > devices.txt ，devices.txt配置包含了android 设备的 ID将被测试时使用。
2. Secondly, you need to write the batches.txt so that it contains the list of the directories containing the fuzzed input media files 
2. 其次，你需要写 batches.txt文件，包含要被 fuzzed的输入 media文件
### Running a fuzzing campaign
### 运行一个 fuzzing 活动
Having configured these two files you can start the fuzzing campaign by issuing the following command:
配置完这两个文件，你可以开始 fuzzing测试了，使用下面的命令：
```
 python test.py stagefright <video|audio> <play|noplay> <index>
    <video|audio> the media batches tested are audio or video files
    <play|noplay> in case of audio testing, try to also test the playback functionality of the framework or not
    <index>       in case you stop the fuzzing campaign at a certain index, you can restart from that certain point (for new campaigns use 0)
```

During the fuzzing process, a separate log will be created for each device in the testing infrastructure. The logs are updated real-time so you can check out partial results during the actual testing.
fuzzing 运行过程中，在测试架构中将为每个设备创建一个独立的日志文件。日志被实时更新所以在实际测试过程中你可以检查结果
### Running a bug triage campaign
### 运行一个 bug分类 程序
The triage mechanism will take the generated logs from the actual fuzzing phase, identify the crashing test cases, resend them to the devices, check if the issues have been encountered before and store the unique bugs. Before starting the actual triage process, you need to copy the generated logs to the root directory of the triage scripts. Also you need to populate the logs.txt config file with the file names of the logs, one per each line.
分类机制将作用在真实的fuzzing日志上，区分crash测试的结果，再次发送他们到设备上，检测是否issues之前已经发生过了并且存储成不同的 bug,在开启真实区分进程前，你需要复制已经生成的日志们到分类脚本的根目录下，你也需要在 log.txt 配置文件的每一行前添加 logs 的文件名。
To start the triage process you need to issue the following command:
开启筛分进程，你需要开启下面的命令：
```

python triage.py <SIGSEGV|SIGILL|SIGFPE|all> <video|audio>
    <SIGSEGV|SIGILL|SIGFPE|all> - type of signal to look out for
    <video|audio>               - the media batches that were tested are audio or video files
```

### Some results - vulnerabilities discovered
### 一些结果--已被发现的漏洞
- Multiple integer overflows in Stagefright code (libstagefright SampleTable):
- 在 stagefright 代码多个整数溢出
	- CVE-2014-7915 

	- CVE-2014-7916 

	- CVE-2014-7917

- A crafted MPEG4 media file can result in heap corruption in libstagefright, that can lead to arbitrary code execution in the mediaserver process:
- 一个精心 MPEG4 媒体文件能导致任意访问 libstagefright 的堆内存，这能够导致任意代码执行在 mediaserver 进程：
	- CVE-2015-3832

### Papers, presentations
### 论文，报告
Android Builders Summit, March 2015 - http://events.linuxfoundation.org/sites/events/files/slides/ABS2015.pdf 

#### Notice
#### 注意
This software is a prototype and it was developed in a specific environment. Don't expect everything to work out of the box.
