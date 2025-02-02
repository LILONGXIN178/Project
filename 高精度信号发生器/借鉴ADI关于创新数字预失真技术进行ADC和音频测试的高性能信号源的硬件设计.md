# 借鉴ADI关于创新数字预失真技术进行ADC和音频测试的高性能信号源的硬件设计



**简述：要测试精密仪器仪表，需要使用超低失真、低噪声、高性能的信号发生器。此外，加入一种创新数字预失真算法可以进一步增强测试信号的保真度，从而以低成本的小尺寸实现出色的低失真信号。**



## 高性能混合信号测试需求

​	现代ADC和其他混合信号器件经常需要使用一个源来测试高性能直流和交流特性。在所有情况下，源的性能都必须优于被测设备(DUT)的性能。

​	执行直流测试是为了确保无失码，并且验证差分非线性(DNL)、积分非线性(INL)、偏置和增益误差。这些测试需要利用低噪声和高分辨率的直流耦合单发线性信号（例如斜坡信号）来表征INL和DNL性能。在这种类型的测试中，需要达到高分辨率，以便执行ADC中的所有可用代码。

​	交流测试验证总谐波失真(THD)、信纳比(SINAD)和无杂散动态范围(SFDR)等参数。这些测试通常使用超高质量的信号音（正弦波）进行，这意味着，其中不能包含高于目标规格的任何谐波成分。为了完成这项任务，测试工程师可以采用定制的滤波器来消除测试信号中不需要的失真产物，但这会增加系统的复杂性和成本。但是，来自源的宽带噪声很难在相关信号周围进行滤波。来自源的噪声需要低于被测ADC的本底噪声，确保不会降低预期的测量目标。



## 低失真设计的关键考虑因素：分辨率和线性度

​	失真可以表示为在任何给定点上信号幅度的误差。这些误差导致信号偏离其理想的信号形状。对于数字合成信号，想要准确表示相关信号的每个样本，关键在于采用真正的高分辨率DAC，保证线性度达到最低有效位(LSB)。由于INL和DNL是量化转换器与其理想转换函数之间的偏差的指标，这些线性度误差会直接影响到高保真信号的再现。

​	由于周期信号的失真通常用THD表示，我们需要量化分辨率和INL对THD的影响，以选择合适的精密DAC。为了观察低THD，需要采用低本底噪声，这意味着需要高信噪比(SNR)。从根本上说，转换器的信噪比受到量化噪声的限制。一般认为，信噪比和分辨率的关系表达式如下所示
$$
SNR=6.02 N+1.76+10\times log \left(\frac{f_S}{2\times BW}\right)(dB)
$$
​	其中N为转换器中可用的位数，fs为采样率，BW为测量带宽。例如AD7606我们所需的信噪比至少要优于95.5 dB，最好是其3倍，约为100 dB，那么在100 dB信噪比时，所需的分辨率约为16位。

## 信号产生路径的设计

​	要设计一个能够满足失真和噪声要求的源，首先需要几个关键组件：DAC和其基准电压电路。可以使用 AD5761这一16位精密DAC达成这一目标。它的高分辨率和线性度优于2 LSB，保证在使用5V输出电压时，能够以高准确度再现误差小于80 μV的信号电平。

​	输出信号路径的简化示意图如图2所示。两个R-2R的精密DAC采用相反的极性来实现全差分路径，进一步提高信噪比，并从接地引起的串扰中解耦相关信号。低噪声基准电压源（例如 [LTC6655](https://www.analog.com/cn/products/ltc6655.html)）和 [AD8676](https://www.analog.com/cn/products/ad8676.html) 精密运算放大器结合，提供每个R-2R型DAC的高线性双极运行所需的正负基准电压电平。具体框图如下所示：

![](https://raw.githubusercontent.com/LILONGXIN178/img/master/image-20241018111152220.png)

​	由于	R-2R型DAC所使用的高精度结构，故在使用精密DAC生成信号时，遇到的常见挑战在于代码转换期间生成的毛刺能源。毛刺会使生成的信号的时域特征变形，给DUT提供多余的能量。对于周期信号，这些毛刺会在频域中产生与基频信号音谐波相关的杂散成分。要解决这一问题，可以对毛刺能量进行滤波，这会大大降低信号带宽和源的建立时间。有一种更好的解决方案是基于采样保持电路实施去毛刺电路，且采用低电荷模拟注入开关。由于此次项目中对带宽要求相对较窄，故此次仅使用高阶低通滤波电路。

​	DPD算法可在后续算法/软件设计中再次讨论，本篇记录仅为对硬件方面设计处理。

## 混合信号发生器的主要器件选型

### R-2R型DAC：

#### 1.AD5541(完全16位R-2R型DAC)

- **3 V和5 V单电源供电**
- **低功耗：0.625 mW**
- **建立时间：1 μs**

- **SPI/QSPI/MICROWIRE兼容接口标准**
- **上电复位可将DAC输出清零至0 V（单极性模式）**
- **低毛刺：1.1 nV-s**
- **保证单调性：±1 LSB（最大值）**

#### 2.AD5761(集成2 PPM/°C基准电压源的多范围、16位、双极性电压输出型DAC)

- **低漂移2.5 V基准电压源：±2 ppm/°C（典型值）**
- **总非调整误差(TUE)：0.1% FSR（最大值）**
- **16位分辨率：±2 LSB（最大值），积分非线性(INL)**
- **保证单调性：±1 LSB（最大值）**
- **单通道、16/12位DAC**
- **建立时间：7.5 μs（典型值）**
- **集成基准电压缓冲器**
- **低噪声：35 nV/√Hz**
- **低毛刺：1 nV- sec（0 V至5 V范围）**

#### 3.AD5781（18位电压输出DAC，±0.5 LSB INL、±0.5 LSB DNL）

- **单通道18位DAC，INL = ±0.5 LSB**
- **噪声谱密度：7.5nV/√Hz**
- **长期线性稳定性：0.05 LSB**
- **温度漂移：<0.05 ppm/°C**
- **建立时间：1μs e**
- **毛刺脉冲：1.4 nV-s**
- **工作温度范围： -40 °C至125 °C**



​	此处选用AD5541进行V1.0设计，此款DAC在保持性能较高的同时，有着引脚较少，且成本较低的优点。可作为V1.0测试方案所使用。后期改进可使用AD5781进行性能上的提升优化。

![](https://raw.githubusercontent.com/LILONGXIN178/img/master/image-20241021200724251.png)



### 基准源芯片：REF5025

2.5V、3µVpp/V 噪声、3ppm/°C 温漂、精密串联电压基准(相对来说，成本较低，且性能较优)

![](https://raw.githubusercontent.com/LILONGXIN178/img/master/image-20241018195402899.png)

![image-20241021204143110](https://raw.githubusercontent.com/LILONGXIN178/img/master/image-20241021204143110.png)

### 全差分六阶巴特沃斯滤波器：ADA4945（高速、±0.3µV/°C 失调漂移、全差分 ADC 驱动器）

- **宽电源电压范围：3 V至10 V**
- **宽输入共模电压范围： −VS至 +VS − 1.3 V**
- **轨到轨输出**
- **采用额定双电源工作模式**
  - **全功率模式：4 mA (145 MHz)**
  - **低功率模式：1.4 mA (80 MHz)**
- **全功率模式**
  - **低谐波失真**
  - **−133 dBc HD2和−140 dBc HD3 (1 kHz)**
  - **−133 dBc HD2和-116 dBc HD3 (100 kHz)**
  - **快速建立时间**
    - **18位：100 ns**
    - **16位：50 ns**
  - **输入电压噪声：1.8 nV/√Hz，f = 100 kHz**

- **最大失调电压：±115 μV（−40°C至+125°C）**

​	按照ADI推文中的芯片进行设计使用ADA4945芯片，进行全差分的滤波器设计。
​	使用FliterPro进行全差分六阶巴特沃斯滤波器进行设计：

![](https://raw.githubusercontent.com/LILONGXIN178/img/master/image-20241021200852653.png)	![](https://raw.githubusercontent.com/LILONGXIN178/img/master/image-20241021201007292.png)



### **DPD数字预失真信道（V1.0对该项不做规划）**



​	故信号通路整体信道设计如下：

![](https://raw.githubusercontent.com/LILONGXIN178/img/master/image-20241021211926128.png)

## 电源树设计

​	由于该设计整体所需电源为+5V，+5V_A，-5V_A。并且为了保持单板的电源稳定性。故选用一款双路DC-DC输出正负电压后，再通过LDO转出较为纯净的供电电平。

### **双路DC-DC选型：**TPS65131

具有双通道正负输出（750mA 典型值）的双电源转换器

**• 输入电压范围：2.7V 至 5.5V**
 **• VPOS 正升压转换器输出**
	 – 可调节输出电压：高达 15V
	– 两种开关电流限制选项：0.8 A 和 2 A
	– 转换效率高达 89%
 **• VNEG 负反相降压/升压转换器输出**
	– 可调节输出电压：低至 -15V
	– 两种开关电流限制选项：0.8 A 和 2 A
	– 转换效率高达 81% 
**• 外部 P 沟道 FET 的控制输出支持与电池连接完全断开** 
**• 1µA 关断电流 • 用于灵活输出时序的单个使能输入** 
**• 保护特性**
	– VPOS 和 VNEG 过压保护
	– 输入欠压闭锁
	– 热关断保护

![](https://raw.githubusercontent.com/LILONGXIN178/img/master/image-20241022102048680.png)

### 运放正负压供电LDO选型：LT3032

双通道、150mA、正 / 负、低噪声、低压差线性稳压器

- **低噪声：20μVRMS (正) 和 30μVRMS (负)**
- **低静态电流：每通道 30μA** 
- **宽输入电压范围：±2.3V 至 ±20V**
- **输出电流：±150mA**
- **低停机电流：总值 <3μA (典型值)**
- **低压差电压：每通道 300mV**
- **固定输出电压: ±3.3V、±5V、±12V 和 ±15V**
- **可调输出从 ±1.22V 至 ±20V**
- **无需保护二极管**
- **采用 2.2μF 输出电容器可稳定**
- **采用陶瓷电容器、钽电容器或铝电容器可稳定**
- **可在反向输出电压条件下起动**
- **电流限制和热限制**

![](https://raw.githubusercontent.com/LILONGXIN178/img/master/image-20241022105559235.png)

故电源整体设计如下：	

![](https://raw.githubusercontent.com/LILONGXIN178/img/master/image-20241022110157357.png)



## PCB如下：

![](https://raw.githubusercontent.com/LILONGXIN178/img/master/image-20241022162820232.png)
