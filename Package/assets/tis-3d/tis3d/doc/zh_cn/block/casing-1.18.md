# 外壳
![在逻辑的框架下](item:tis3d:casing)

外壳可以安装最多六（6）个模块，每面可装有一个。由于外壳必须要连接到[控制器](controller.md)才能发挥作用，所以通常只能装五（5）个；外壳与外壳之间，或外壳与[控制器](controller.md)之间的面不能连接模块，这些面需用于内部通讯。在装有模块的面上放置外壳或[控制器](controller.md)将强制弹出模块。

外壳为每个安装的模块提供了四个端口，端口可跨外壳边传输数据。如果该边另一侧是外壳，则会跨连接面将数据传输至相邻外壳的相邻端口处。如果另一侧无外壳，则会跨边将模块与同外壳相邻模块相连。

这意味着端口必定能连接到某个模块插槽。但是如果该插槽并未安装模块，则该端口的读取操作和写出操作都不会成功。

外壳可以被[钥匙](../item/key.md)锁定，锁定以后不能安装或者移除模块。可用于防止他人的恶意操作或者意外移除模块。

另外，潜行时[钥匙](../item/key.md)可以用来打开或关闭外壳面上的接收端口。这会停止相邻模块对这个端口的写入操作，也会阻止全方向写操作写到这个端口。这能让你电脑的构造更紧凑，还能节约一些用来直接连接的[执行模块](../item/execution_module.md)（例 如：把从[红外模块](../item/infrared_module.md)接收到的数据直接传输给[红石模块](../item/redstone_module.md)）。
