## script使用方法
### 注意
使用前需确认内核中是否包含ftrace，查看`/sys/kernel/debug/tracing`以确认。
### 概述

script文件夹结构如下
```
|- script
	|- probes.sh
	|- trigger.sh
```
`probes.sh`追踪程序中设置的所有tracepoint以及usb-host中的tracepoint
`trigger.sh`只追踪`getImuDataFrame`读取成功后的tracepoint
程序编译完成后，运行下列指令
```shell
objdump -t -C imu_test | grep _dummy
objdump -t -C imu_test | grep onLive
```
从输出中提取以下内容
```
000000000012ecd0 g     F .text	00000000000000b0              xda_dummy(int)
000000000012c550 g     F .text	00000000000000b8              trigger_dummy()
000000000012ed80 g     F .text	00000000000000b0              spin_dummy(int)
000000000018d9bb g     F .text	000000000000000f              parser_dummy()
00000000001b025c g     F .text	0000000000000010              read_dummy(int)
-----------------------------------------------------------------------------
000000000012dcf0 g     F .text	00000000000001d1              XdaCallback::onLiveDataAvailable(XsDevice*, XsDataPacket const*)

```
修改 script/ 文件夹内脚本内容
```shell
echo 'p:trigger /home/bryan/code/xsens_ros_mti_driver-xsens_ros_mti_driver_lib/build/imu_test:0x12c550' >> ${dir}/uprobe_events

修改所有tracepoint中程序绝对路径以及对应的offset
```
