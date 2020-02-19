---
title: "Raspberry Pi 使用DS18B20温度传感器"
date: 2020-02-11T14:16:06+08:00
tags: ["物联网"]
---


raspberry 温度采集
<!--more-->

# DS18B20 温度传感器
最近心血来潮的想要折腾raspberry，所以就买了一堆的传感器，现在主要讲解温度传感器的使用。

![屏幕快照 2017-05-29 下午10.31.39](http://7xrkms.com1.z0.glb.clouddn.com/屏幕快照 2017-05-29 下午10.31.39.png)

![](http://7xrkms.com1.z0.glb.clouddn.com/14960683305689.png)

Pin1(GND)接到 P1-06(GND)
Pin2(DQ)接到 P1-07(GPIO4)
Pin3(VCC)接到P1-01(3.3v)

为了能够让系统正确的识别传感器，我们需要对**/boot/config.txt**文件进行编辑
> sudo vim /boot/config.txt

在文件的未必添加下面的语句
> dtoverlay=w1-gpio,gpiopin=4

添加完成之后我们重启raspberry
> sudo reboot

重启完成之后我们需要进行/sys/bus/w1/devices目录
> cd /sys/bus/w1/devices

通过调用ls命令，我们发现这个目录会有类似28-0416b3b833ff这样的一个目录，这个目录名称并是你传感器的ID，.在目录下面会有一个w1_slave文件，我们通过调用 cat命令可以读取到传感器的实时数据。
> cd 28-0416b3b833ff
> ls
> cat w1_slave

完整的命令如下图所示
![屏幕快照 2017-05-29 下午10.42.01](http://7xrkms.com1.z0.glb.clouddn.com/屏幕快照 2017-05-29 下午10.42.01.png)
其中t=30312是我实际测量到温度值，在上面的图中的温度为30.312度


同时我们可以利用python脚本来读取数据

``` python
#!/usr/bin/python
def gettemp(id):
  try:
    mytemp = ''
    filename = 'w1_slave'
    f = open('/sys/bus/w1/devices/' + id + '/' + filename, 'r')
    line = f.readline() # read 1st line
    crc = line.rsplit(' ',1)
    crc = crc[1].replace('\n', '')
    if crc=='YES':
      line = f.readline() # read 2nd line
      mytemp = line.rsplit('t=',1)
    else:
      mytemp = 99999
    f.close()
 
    return int(mytemp[1])
 
  except:
    return 99999
 
if __name__ == '__main__':
 
  # Script has been called directly
  id = '28-0416b3b833ff'
  print "Temp : " + '{:.3f}'.format(gettemp(id)/float(1000))
```

我们通过命令行调用，得到如类似下面的结果

![屏幕快照 2017-05-29 下午10.48.50](http://7xrkms.com1.z0.glb.clouddn.com/屏幕快照 2017-05-29 下午10.48.50.png)