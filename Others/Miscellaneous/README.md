## script使用方法
程序编译完成后，运行下列指令
```shell
objdump -t -C imu_test | grep _dummy
objdump -t -C imu_test | grep onLive
```
从输出中提取以下内容
```
000000000012ecd0 g     F .text	00000000000000b0              xda_dummy(int)
000000000012c5b0 g     F .text	00000000000000b8              trigger_dummy()
000000000012ed80 g     F .text	00000000000000b0              spin_dummy(int)
000000000018d9bb g     F .text	000000000000000f              parser_dummy()
00000000001b025c g     F .text	0000000000000010              read_dummy(int)
-----------------------------------------------------------------------------
000000000012dcf0 g     F .text	00000000000001d1              XdaCallback::onLiveDataAvailable(XsDevice*, XsDataPacket const*)

```
修改 script/ 文件夹内脚本