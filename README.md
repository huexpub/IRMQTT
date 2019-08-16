
![enter image description here](https://github.com/huexpub/IRMQTT/blob/master/pic/irmqtt.jpg?raw=true)

## 声明：
1. IRMQTT 是一个开源项目，源代码来自https://github.com/crankyoldgit/IRremoteESP8266，
2. IRMQTT PCB你可以自由的使用，并且制作分享与朋友交流，但严禁商业行为!

IRMQTT首贴：
https://bbs.iobroker.cn/thread-176-1-1.html

IRMQTT模块贴：
https://bbs.iobroker.cn/thread-221-1-1.html

IRMQTT支持列表：
https://github.com/crankyoldgit/IRremoteESP8266/blob/master/SupportedProtocols.md#ir-protocols-supported-by-this-library

## 准备工作：
此次PCB模块为适应更高的通用性，以苹果绿点5V1A充电器为供电对象，最小化PCB板和硬件体积，合体后仅增加7.2mm的高度，一体感非常好!  由于体积定位，元件大量采用0402贴片，PCB面积仅21*20MM。

### PCB
本次PCB文件单体制造难度较大，新手尽量转移到ESP12F项目和ESP01F改装模块！高手可进

项目地址: https://github.com/huexpub/IRMQTT

* 分支包含ESP01F文件和ESP12F文件，默认为IRMQTT USB公插版本，
* 固件目前测试可以通用，如果内部为ESP01F，会有OTA文件限制！
* 刷机均需要1.27夹具或者焊接线材

### 贴装
PCB贴装，请控制好你的烙铁温度，保持300以内，针对发射贴片，请避免长时间操作

![enter image description here](https://github.com/huexpub/IRMQTT/blob/master/pic/pcb.jpg?raw=true)

* 保持发射贴片丝印标记正确，除中心发射元件，其它保持45度角以得到更好的发射范围
* 由于接收元件外露在3D外壳之外，请确保元件正中，否则外壳无法安装

### 刷机

由于空间较小，刷机TTL为4P*1.27MM 故，你需要特殊的工具，比SOP8，SOP16刷机夹，1.27测试针，或者自己焊接线刷机，刷机后，后期均由OTA完成更新！PCB正面，FP脚为0-GND的短接点，短接后，ESP进入下载模式，刷机完成后，请去除短接！

### 3D
 3D外壳为上、下沉方式，需要M2沉头螺丝两颗，目前设计的螺丝帽直径最大为4.5MM 沉头内六角螺丝。
 
![enter image description here](https://github.com/huexpub/IRMQTT/blob/master/pic/irmqtt-stl.png?raw=true)

## 安装使用：
软件采用WIFIMANAGER配网，上电后，搜索IRMQTT热点，连接上，转入配网WEB页面，未弹出请尝试访问192.168.4.1地址

### 配网

>请务必保证MQTT服务器地址正确，否则可能无法进入WEB页面，并且无法重新配置！


![enter image description here](https://github.com/huexpub/IRMQTT/blob/master/pic/irmqtt-wifi.gif?raw=true)

### 使用
进入配置的IRMQTT设备IP WEB，页面如下：

![enter image description here](https://github.com/huexpub/IRMQTT/blob/master/pic/irmqtt-web.gif?raw=true)

选择对应的空调型号，模式，电源，温度，并点击 UPDATE/SEND，空调有反应后，视为有效型号，若无反应，请对应尝试model 1-6子设备型号，如均无反应，可能不被支持。

### 反馈
如果知道IRMQTT模块是否已经能正常的反馈呢，你可以使用MQTT工具监听。如下图GIF演示！

![enter image description here](https://github.com/huexpub/IRMQTT/blob/master/pic/irmqtt-mqtt.gif?raw=true)

*  默认连接上MQTT后，出来的型号和设备等是默认生成的，不是接收到的
* 操作之后能出现ON OFF和有效的数据源，视为可反馈，
* 关于监听工具，你可以使用docker 镜像  huex/webzsh:armbian (https://mysensor.pub/gateway/webzsh/)

### 接入
由于采用标准MQTT协议，并且作者以Homeassistant为蓝本输出消息，故，你可以使用IRMQTT-WEB控制台发送 MQTT discovery 让平台自动发现即可，也可以手动配置，配置如下：


```

climate:
  - platform: mqtt
    name: zwaircon
    modes:
      - "off"
      - "auto"
      - "cool"
      - "heat"
      - "dry"
    fan_modes:
      - "auto"
      - "min"
      - "low"
      - "medium"
      - "high"
      - "max"
    swing_modes:
      - "off"
      - "auto"
      - "highest"
      - "high"
      - "middle"
      - "low"
    power_command_topic: "ir_server/ac/cmnd/power"
    mode_command_topic: "ir_server/ac/cmnd/mode"
    mode_state_topic: "ir_server/ac/stat/mode"
    temperature_command_topic: "ir_server/ac/cmnd/temp"
    temperature_state_topic: "ir_server/ac/stat/temp"
    fan_mode_command_topic: "ir_server/ac/cmnd/fanspeed"
    fan_mode_state_topic: "ir_server/ac/stat/fanspeed"
    current_temperature_topic: "tele/bksensor/SENSOR"
    current_temperature_template: "{{ value_json['SI7021'].Temperature }}"
    swing_mode_command_topic: "ir_server/ac/cmnd/swingv"
    swing_mode_state_topic: "ir_server/ac/stat/swingv"
    min_temp: 16
    max_temp: 30
    temp_step: 1
    retain: false
    
```
*  current_temperature_topic 和 current_temperature_template 为外部引用温度，这里采用了JSON格式的消息
*  请依据你的空调实际情况修改对应的菜单


### OTA
由于ESP01F flash只有1MB，实际固件不能超过467KB，但由于按开源编译，固件至少490K，造成由于空间不够无法在线更新，故我们只能曲线升级，在不破坏SPIIFS的情况下，先上传一个只有WEB-UPDATE功能的迷你型固件，该固件也采用WIFIMANAGER配置 网，并且能继承原有配置，升级后重新打开IP，进入WEB，选择最新的IMQTT固件升级即可

总结即： OTA--MINI 固件--正常固件

以下为演示：
![enter image description here](https://github.com/huexpub/IRMQTT/blob/master/pic/irmqtt-ota.gif?raw=true)
  
## SMT开车
  此次为了方便各位，目前已经将PCB 容阻元件、三极管、LED、LDO 采用 SMT 小批量产，到手只需要焊接发射元件、接收元件、USB、天线贴片、ESP01，极容易操作的元件部分！方便能DIY并且有3D打印机的小伙伴!

 另外为了方便不原折腾的人，需要上成品车的伙伴，你可以直接从淘宝购入成品，并且支持7天无理由和自购运费险，如果不能使用，你可以退货，承担来回运费即可！

链接：https://item.taobao.com/item.htm?spm=a1z10.1-c.w4023-7677041950.2.57033a0cMouRqm&id=600537216286

*  成品涉及到3D打印，由于打印消耗时间和人工，故发货时间7天内
*  淘宝不经常在线，如果你有疑问和好的建议尽量在论坛或者ioBroker群MYSENSOR群@HUEX


## Q&A

- 1.购买单PCB文件，我还要购买哪些材料？
A: 安信可ESP01F 、BOM表中的EVERLIGHT台产亿光 IR26-61C 发射贴片、接收元件H638T ，2.4G天线RFANT3216-Z。

- 2.空调在列表中，但无法发射？
A: 请先使用MQTT监测是否遥控有接收到数据，如果有数据，那么你发射的型号可能不对，特别是像美的，格力，等在海外有代工或者授权品牌，采用的，你可能需要使用其它品牌型号，比如COOLIX、ELECTRA_AC等。如果你接收都没有任何消息，那么很遗憾，至少源码暂时可能无法支持。

- 3.部分操作码不正常？
A: 出现这个情况，你可以尝试更换其它MODEL试试，1-6每个都试一下，如果问题依然，那么需要向作者提交ISSUES来解决，你需要提交原始码，原始操作，有差异化的信息，然后一并提交，具体提交办法你可以参考作者源码的入口。

- 4.有支持其它电源吗？
A. 理论上体积大于25*25mm的电源都可以3D画图解决，但手头暂时无其它电源，也没有额外的精力设计制作，暂时不扩充，你可以自行设计！

- 5.固件我可以自己编译上传吗？
A.没有问题，按上述OTA操作，先上传MINI固件，然后上传最新的即可。