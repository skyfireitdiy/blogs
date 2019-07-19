# python 多进程

python 多进程可以充分使用多核资源，而且实现起来比较简单。

## 多进程

```python
import os
from multiprocessing import Process


def sum(f, t):
    s = 0
    for i in range(f, t):
        s += i
    print("Subprocess pid:", os.getpid(), s)


def main():
    print("Main pid:", os.getpid())
    p = Process(target=sum, args=(0, 100))
    p.start()
    p.join()


if __name__ == '__main__':
    main()
```

程序输出：

```text
Main pid: 6864
Subprocess pid: 11284 4950
```

上面程序中，sum 运行在子进程中。


## 进程池

```Python
from multiprocessing import Pool
import os
import time


def func(start, end, n):
    for i in range(start, end):
        time.sleep(1)
        print(n, i)


def main():
    p = Pool(4)
    for i in range(10):
        p.apply_async(func, (i*10, (i+1) * 10, i))
    p.close()
    p.join()


if __name__ == '__main__':
    main()
```

对 Pool 对象调用 join() 方法会等待所有子进程执行完毕，调用join()之前必须先调用 close() ，调用 close() 之后就不能继续添加新的 Process 了。

在上一个程序中，允许最多同时运行 4 个进程。

## 多进程通信

如果需要多个进程进行通信，可以借助 Queue。

```python
import os
from multiprocessing import Process, Queue


def sum(f, t, q):
    s = 0
    for i in range(f, t):
        s += i
    print("Subprocess pid:", os.getpid())
    q.put(s)


def main():
    print("Main pid:", os.getpid())
    q = Queue()
    p = Process(target=sum, args=(0, 100, q))
    p.start()
    s = 0
    for i in range(100, 200):
        s += i
    print("0+1+2+...+198+199=", s + q.get())


if __name__ == '__main__':
    main()
```

程序输出：

```text
Main pid: 13496
Subprocess pid: 5152
1+2+...+198+199= 19900
```

上面的程序完成计算 0+1+2+...+198+199 的值，主进程计算 0 到 99，子进程计算 100 到 199，然后子进程将计算结果放入队列，主进程读取队列中的数据，并与自身的结果相加。

## 子进程

有的时候我们需要调用其他的程序，并发送输入，获取输出结果。比如我们需要运行 cmd.exe，然后输入

```text
wmic
cpu
exit
exit
```

查看输出结果。

```python

```

在我本机输出如下：

```text
Microsoft Windows [版本 10.0.18362.239](c) 2019 Microsoft Corporation。保留所有权利。D:\360MoveData\Users\skyfire\Desktop\test>wmic
wmic:root\cli>AddressWidth  Architecture  AssetTag     Availability  Caption                               Characteristics  ConfigManagerErrorCode  ConfigManagerUserConfig  CpuStatus  CreationClassName  CurrentClockSpeed  CurrentVoltage  DataWidth  Description                           DeviceID  ErrorCleared  ErrorDescription  ExtClock  Family  InstallDate  L2CacheSize  L2CacheSpeed  L3CacheSize  L3CacheSpeed  LastErrorCode  Level  LoadPercentage  Manufacturer  MaxClockSpeed  Name                                     NumberOfCores  NumberOfEnabledCore  NumberOfLogicalProcessors  OtherFamilyDescription  PartNumber   PNPDeviceID  PowerManagementCapabilities  PowerManagementSupported  ProcessorId       ProcessorType  Revision  Role  SecondLevelAddressTranslationExtensions  SerialNumber  SocketDesignation  Status  StatusInfo  Stepping  SystemCreationClassName  SystemName       ThreadCount  UniqueId  UpgradeMethod  Version  VirtualizationFirmwareEnabled  VMMonitorModeExtensions  VoltageCaps
64            9             Fill By OEM  3             Intel64 Family 6 Model 58 Stepping 9  4                                                                 1          Win32_Processor    3201
         101             64         Intel64 Family 6 Model 58 Stepping 9  CPU0                                      100       205                  1024                       6144         0
                 6      7               GenuineIntel  3201           Intel(R) Core(TM) i5-3470 CPU @ 3.20GHz  4              4                    4                                                  Fill By OEM                                            FALSE                     BFEBFBFF000306A9  3              14857     CPU   TRUE                                                   SOCKET 0
   OK      3                     Win32_ComputerSystem     DESKTOP-HOI5DJO  4                      36                      TRUE                           TRUE

wmic:root\cli>
D:\360MoveData\Users\skyfire\Desktop\test>
return 0
```
