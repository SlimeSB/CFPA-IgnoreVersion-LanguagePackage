# 磁带存储模块

![都“带”上就完事了！](item:tisadvanced:tape_storage)

磁带存储模块提供了一盘可读可写的8KiB磁带，用于存储数据。其读写头位置与所存储数据在计算机重启后会保留。可以卸载模块并将其重新安装到另一台计算机上，其状态将保持不变。

读取操作将返回读写头当前位置的字节的值。写入操作可以覆写读写头当前位置的值，或者向某一方向卷动磁带。写入操作接受一个16位的值，其高8位被解释为用于控制磁带的控制码；低8位则是控制码使用的附加数据值。各控制码的功能如下所示：

## 控制码

- `0`：向磁头当前位置写入数据
- `1`：将磁带向前卷动一字节，并丢弃附加数据
- `2`：将磁带向后卷动一字节，并丢弃附加数据
- `3`：将磁带向前卷动与附加数据值相等的字节数
- `4`：将磁带向后卷动与附加数据值相等的字节数
- `5`：将磁带卷动到其开始位置，并丢弃附加数据
- `6`：将磁带卷动到其结束位置，并丢弃附加数据
