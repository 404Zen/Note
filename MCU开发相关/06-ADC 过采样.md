这一篇是企图对以下内容的理解。
ST给出的ADC过采样例程 [STSW-STM32014](https://www.st.com/zh/embedded-software/stsw-stm32014.html) 
以及对应的应用笔记 [AN2668 Improving STM32F1 Series, STM32F3 Series and STM32Lx Series ADC resolution by oversampling](https://www.st.com/resource/en/application_note/an2668-improving-stm32f1-series-stm32f3-series-and-stm32lx-series-adc-resolution-by-oversampling-stmicroelectronics.pdf)

# 介绍
过采样适用于需要提升ADC精度的场合，对输入信号进行过采样和抽取的概念，可以去掉对外部ADC的需求，并降低应用消耗。

在AN2668这本手册中介绍了两种提升ADC精度的解决方案，这两种方案基于同样的原理：使用最大的ADC采样频率(1MHz)对输入信号进行采样，抽取其中的信号以提高分辨率。

应用笔记分为两部分，第一部分介绍怎样进行过采样提升 ADC分辨率，第二部分介绍了在 STM32F1系列、STM32F3系列以及STM32Lx系列单片机上的具体实施过程与嵌入式软件流程图。

# 定义 ADC 信噪比

ADC 是对一个模拟信号进行一个有限的数字化过程。因为数字领域需要使用一个有限的数字表示一个连续的信号，这个转换过程就引入了 ADC 输入范围和 量化误差。

对于一个理想的 ADC， 其量化误差在 ±0.5 LSB 之间。在采样到的样本信号是非常多变的，并且采样率与输入频率不同步的情况下，量化误差可以认为是白噪声，其能量从直流域均匀分布到一半的采样频率。有关其密度计算的更多信息，参阅附录A 量化误差章节。

--------------------------------------

# 奈奎斯特定理和过采样

奈奎斯特定理指出，想要还原模拟输入信号，必须以大于输入信号最大频率分量的两倍速率对信号进行采样。

不符合奈奎斯特定理将会导致混叠效应，并且无法从输入样本中完全重建模拟信号。因此，对于大多数应用，ADC输入端需要一个低通滤波器滤除低于采样频率一般的频率。低采样频率将难以处理滤波器约束？(It is difficult to handle the filter constraints with low sampling frequencies)

过采样包括**以高于奈奎斯特频率限制的速率对模拟信号进行采样，过滤样本并通过抽样降低采样速率**。使用这种方式降低了 抗混叠低通滤波器 的限制。


------------------------

# 使用白噪声进行过采样

## 具有白噪声输入的过采样信号的信噪比

假设量化噪声被同化为一个白噪声，其功率密度均匀地分布在直流与二分之一奈奎斯特频率之间。这个功率密度和采样频率无关。
当提升这个采样频率的时候，量化噪声分布在整个采样频率的带宽上。
![](Pasted%20image%2020230319151936.png)
根据 Figure 1，当以一个更高的采样频率进行采样时，灰色部分所表示的同样的噪声能量分布在等于采样频率的带宽上，该频率的远大于信号带宽f<sub>m</sub>，总的噪声功率中只有相对较小的一部分分布在[-fm, fm]频带内，信号频带外的噪声功率可以通过一个数字低通滤波器进行大幅度衰减。
(提高采样率，噪声能量不变，并且平均分布在更宽的范围，从而噪声的幅值降低。原始信号没变，但是噪声赋值减少，也就是信噪比提高了。)

降低量化噪声可以提升信噪比，从而提升ADC的有效位数，以一个比奈奎斯特采样频率快 OSR 倍的速率对信号进行采样，得到以下**信噪比**计算公式：
![](Pasted%20image%2020230319152951.png)
(其中，过采样速率OSR=Fs/(2╳BW)，BW为带宽。)
因此，采样频率每增加一倍，带内噪声就会减少3dB，测量分辨率会提升 1/2 位，因此需要 6dB SNR 增益才能为 ADC 增加 1 位的分辨率。

通常情况下，**如果需要增加 p 位分辨率，那么 ADC 的采样频率至少应为**：
$$F_{OVS} = 4^PF_S$$
F<sub>S</sub> 指的是当前 ADC 所使用的采样率。

**提升一位分辨率需要4^1 = 4倍于原来的ADC采样率**。
提升两位则需要 4^2 = 16倍采样率。

-------------------

## Decimation(抽取/抽样？)
在传统的定义里面，平均值是将m个样本的值相加并除以m。对 ADC 采样到的数据进行平均相当于一个减小信号波动的和噪声的低通滤波器。平均法通常用于平滑输入信号和滤除speaks(也许是peaks, 滤除波峰)。

一个普通的求平均值并不能提升ADC的分辨率，因为 m 个 N-bit 的样本之和除以 m  是一个 N-bit 样本的表示。

抽样是另一种平均方法，当与过采样相结合时，抽样可以提升ADC的分辨率。

实际上，添加 4<sup>p</sup> 倍 ADC N-bit 样本，就可以给出一个信号在 N+2p bits 上的表示。为了有添加 p 个有效位，总和需要右移p位。

这种具有相等滤波系数的 FIR(Finite Impulse Response) 滤波器 能够让用户 从OSR(OverSample Rate) 输入样本中给定一个输出样本的计算来滤除过采样频率。
This FIR filter with equal filter coefficients enables the user to filter the oversampling frequency by giving an output sample computed from the OSR input samples.

过采样的方法限制的最大的输入频率带宽。对于STM32F1系列，其具有1Msps采样率的 ADC， 那么可处理高达 500 KHz 的信号，如果需要提升两位分辨率，当采用白噪声的方式时，输入信号的最大频率将被限制为 500 / 16 = 31.25 KHz 。

--------

## When is this method efficient？(这种方式啥时候才有效？)
为了使过采样和抽样这种方式正常工作，必须满足以下条件：
- 输入信号中存在噪声，并且这个噪声必须在所关注的近似频段上具有均匀功率密度分布。
- 这个噪声的幅度必须能使采样信号随机产生1LSB 以上的变化。否则，如果输入信号一样，平均求和将得不到任何额外的分辨率提升。对于大多数应用，内部ADC的热噪声和输入信号噪声足以使用这个方法。如果热噪声没有提供足够的幅度使输入信号产生随机变化，则应在输入信号中添加人工白噪声，这个操作称之为"抖动"。关于这一点，可以提出两个问题，第一个是“如何评估ADC噪声并测试其高斯标准(Gaussian criteria)?”, 第二个是“如何产生所需要的白噪声？”。
	- 一种实用的检测输入信号噪声的高斯标准的方法是 查看 **干净的直流信号的ADC输出结果分布**。直方图的方法可以用于验证噪声是否服从高斯分布。Figure 2.中的实例展示了两种可能的情况。
	 ![](Pasted%20image%2020230319163405.png)
		 - 左边是具有白噪声信号的直方图，左边则是不具有白噪声信号的直方图
	
	 - 在必须要添加外部噪声抖动到输入信号的情况下，可以通过将二极管或者电阻添加到输入信号中以产生一个热噪声
	 - 输入噪声不得与有用的信号相关，并且输入信号应具有相等的概率落在相邻的ADC输出值之间。这意味着，对于使用反馈过程的系统，这个方法无效。


## Method implementation on the STM32F1 Series, STM32F3 Series and STM32Lx Series devices(在单片机上实施这个方法)

此方法描述了在 STM32F1 系列、STM32F3 系列和 STM32Lx 系列器件上实施和测试过采样方法所采取的不同步骤。(接下来只关注F1相关内容)

根据前面的章节，为了使这个方法能有效工作，必须有一些白噪声使得输入信号可以随机变化 1/2 LSB。为此，必须考虑应用环境噪声。

第一步是计算 ADC 白噪声，以确定是否需要将外部白噪声添加到输入信号中。在典型的应用电路中，计算出的白噪声不仅包括ADC内部噪声，也包含了不同的板子的组件以及布局所产生的噪声。因此，这个评估结果取决于实际的电路板，但是方法是相同的。

直方图法用不同的直流电压输入，采样多次这个输入电压(例如5000次)，然后使用电子表格来输出一个直方图。

For example, for a 1.65 V DC input voltage applied on the STM3210B-EVAL evaluation board, the histogram shown in Figure 3 is detected
![](Pasted%20image%2020230319164809.png)
ADC热噪声可以通过这个直方图计算出来。(应用手册中没有提供计算方法，他认为这不是这本手册的重点。T T )

进行ADC噪声测试的操作如下：
- 取消 `oversampling.h` 文件中 `#define Themal_Noise_Measure` 的注释。
- 配置ADC转换次数 `Total_Samples_Number`, 这个值必须小于 65535。DMA 通道配置为存储 N个 ADC 采样结果 到 RAM 中的缓存。 转换结束后，将产生一个中断并计算每个ADC值出现的次数。
- 为了计算ADC值出现的次数，定义了一个变量给出对应的ADC值。

当代码运行时，`Relevant_ADC_Samples` 个 ADC 样本及其对应出现的次数在 HyperTerminal 上
显示，HyperTerminal 配置为 115200，8N1。如果采样数量小于 `Relevant_ADC_Samples`，则ADC值和其出现次数均显示为0。用户可以通过这些数据构建直方图。

	个人理解：这种方式的重点就在于 一组采样结果 是符合高斯正态分布的；通过增加采样次数，使得平均值更接近 实际值。
	更直白一点就是，提高正确结果出现的次数，平均之后的值就更接近正确结果。但是为了提升分辨率，数据就需要进位，那么就需要降采样。
	所以，需要提升一位的分辨率，需要得到4倍于原来的数据，然后将所有数据之和 右移1bit 就是提升了一位分辨率之后的结果。



### Oversampling using a white noise embedded software flowchart(采用白噪声进行过采样的嵌入式软件流程图)

STM32F1 的 片上ADC采样率固定为 1MHz。ADC DMA通道配置为将 **过采样输入样本数量** 个数据 传输到缓存中。此传输配置为发生一次，在DMA传输完成后将会产生中断并计算过采样结果。

采用通用定时器 TIM2 产生输入信号采样频率。在这个例程中，TIM2 参考时钟配置为 1uS。这个时钟周期决定了输入信号的采样频率，可以在 `oversampling.h` 中的 `#define Input_Signal_Sampling_Period` 进行定义。每当 TIM2 的 update interrupt 触发，DMA将被重新使能然后搬运 ADC 转换结果。
**Figure 4 是对以上操作的总结。**
![](Pasted%20image%2020230319170912.png)

过采样的数据将会在DMA传输完成中断中计算。由于同步的原因，建议在第二次TIM2中断时才开始读取数据。

注意，对于这种实现方式，TIM2的周期必须大于ADC转换过采样样本所需的时间，并且大于ADC中断执行的时间。

如果应用所需要的采样频率刚好是 OSR uS, 则用户无需使用TIM2来生成输入采样频率，但是必须将 DMA 配置在连续模式下运行，并且在DMA传输完成中断中做出对应的更新，过采样数据通常在DMA传输完成中断中进行计算。

### Oversampling using white noise result evaluation(使用白噪声评估结果进行过采样)

为了评估过采样方法，用户必须取消注释 `#define Oversampling_Test` 行并配置具有增强分辨率的样本数。
当取消了该行的注释之后，会在RAM中创建一个缓存来存储过采样数据。缓存中的数据会显示在超级终端上。用户可以将这个真实数据与预期结果进行比较。

为了评估新的增强之后的 ADC，将一个频率为 50Hz，幅度为1V 的 斜坡信号输入到ADC中，并使用过采样算法每100uS进行一次采样。

与此方法相关的 嵌入式软件 在 `WhiteNoiseMethod` 文件夹中。

![](Pasted%20image%2020230319172442.png)
![](Pasted%20image%2020230319172453.png)

使用白噪声方法采样一个相同信号的结果如图5和图6所示。图五是增加了一位分辨率的结果，图6是添加了两位分辨率的结果。

When the ramp is sampled without using any extra software resolution, with a 3.3 V reference supply, 1 V corresponds to the digital value 1250. When one additional bit is added, 1 V is sampled as 2500 and when two additional bits are added, 1 V is sampled as 5000. 

**This means that the environment contains enough noise for this method to work.**

-------------------------------

# Oversampling using triangular dither(使用三角抖动的方式进行过采样)
假设输入信号在过采样期间位于两个连续的量化步骤 q0 和 q1 之间，则转换器可以将其转换为 q0 或 q1。 增加额外的p位分辨率意味着确定输入信号在q0和q1之间的相对位置。

通过添加适当的三角信号，量化器生成一系列 q1 和 q0。 对给定间隔内出现的 q1 求平均值可确定输入信号在较低和较高量化步长之间的相对位置。

该理论指出，当输入信号时是一个三角波、ADC采样周期为 OSR times、抖动幅度为 n+0.5 LSB时，可获得最佳结果，其中 n = 0,1,2,3.

这种方法背后的理论相当复杂，因此图 7 作为一个例子来说明这种方法是如何工作的。 在此示例中，ADC 片上分辨率为 3，嵌入式软件添加了三个额外位。 假定输入信号的幅度为 q0+ 0.6LSB（本例中 q0 = 6）。 为了添加三个额外的位，输入信号被采样了 2的2^3 次（16 次）。
![](Pasted%20image%2020230319174925.png)
如果输入信号与三角波形不相关，则证明 SNR 中的增益等于：
![](Pasted%20image%2020230319175145.png)
因此，采样频率每增加一倍，SNR 就会提高 6dB，并增加 1 个 ADC 位分辨率。
通常，为了增加 p 位额外分辨率，过采样频率必须等于：
![](Pasted%20image%2020230319175247.png)
## When does this method work?(这种方法啥时候有效？)
为了使该方法起作用，输入信号在过采样期间的变化不得超过 ±0.5LSB，并且不得与三角抖动信号相关。


## Method implementation on STM32F1 Series, STM32F3 Series and STM32Lx Series devices
为了实施这个解决方案，需要以下内容：
- 一个运放，用于执行输入信号和三角波的求和，为此，需要一个运放构成反相器和加法器？(an op-amp inverter/summing stage is required)。The ST component LMV321 can be used.
- 一个周期为 OSR 乘以 ADC 转换率的三角波。
	用户可以使用信号发生器或片上定时器之一和 RC 网络来生成此三角信号。 实际上，片上定时器会生成占空比从 0 到 100% 不等的 PWM 信号。 可以使用 RC 滤波器对该 PWM 输出进行滤波，以生成从 0 到 VDD 变化的三角信号。 为了产生 0.5LSB 的幅度，输出首先通过一个电容器（以去除直流分量），然后由预分频器 R2/R3 分频（见图 8）。 该预分频器等于 ADC 的字数(This prescaler is equal to the ADC number of words.)。
- 输入信号不得在运算放大器之后改变。 因此，R1 应等于 R3。
- 输入信号和三角抖动的总和被反转。 为此，运算放大器的正输入需要 3.3 V 偏移。 在计算过采样数据之后，减去此偏移量，给输入信号提供一个额外分辨率(this offset is subtracted to give the input signal estimation with an extra resolution)。
![](Pasted%20image%2020230319180608.png)

### Oversampling using triangular dither embedded software flowchart
STM32F1 的 片上ADC采样率固定为 1MHz。ADC DMA通道配置为将 **过采样输入样本数量** 个数据 传输到缓存中。此传输配置为发生一次，在DMA传输完成后将会产生中断并计算过采样结果。

通用定时器TIM2用于产生输入信号采样频率。 为此，TIM2 参考时钟基准配置为 1µS。 它的周期决定了输入信号的采样周期。 它在 `oversampling.h` 文件中由 `#define Input_Signal_Sampling_Period` 定义。

三角抖动是使用配置为 PWM 模式的定时器 TIM3 通过更新捕获比较寄存器 CCR1 生成的。 定时器 TIM3 周期必须等于 ADC 转换速率，CCR1 必须更新 OSR 时间，其中 OSR 是过采样因子。 为此，首先计算可能的 CCR1 值并将其存储到 RAM 缓冲区中，然后使用 DMA 传输更新 CCR1 寄存器，从而无需中断。
请注意，ADC 转换率限制了过采样因子。 例如，在 ADC 以 1 MHz 运行的情况下，STM32F1 系列以 56 MHz 运行。 为了获得 1 µs 的周期，定时器 TIM3 的自动重载寄存器必须等于 55。那么最大只能增加4位分辨率。

当触发 TIM2 更新中断时，ADC 和 TIM3 DMA 将重新启用，转换后的 ADC 值将可用于计算具有更高分辨率位的新样本。 图 9 总结了功能的实现过程：
![](Pasted%20image%2020230319181232.png)
为使该方法起作用，输入信号在过采样期间的变化不得超过 ±0.5LSB。 这意味着对于采用 3.3 V VREF+ 工作的 STM32F1 系列、STM32F3 系列或 STM32Lx 系列设备，过采样期间输入信号的最大允许变化为 ~0.4 mV。

另一方面，对于使用 3.3 V VREF+ 的STM32F1 系列、STM32F3 系列或 STM32Lx 系列时，幅度为 0.5 LSB 的三角波意味着仅有 0.4 mV 幅度。 因此，应用程序环境不能非常嘈杂。 三角波形的任何扰动都会对计算的过采样数据产生影响。

根据实现过程，三角波是通过 STM32和一个滤波器消减这个 1 MHz 方波生成的。 集成定时器 PWM 输出信号以提供幅度为 3.3 V 的三角波信号。 除法是用 R3/R2 的比率完成的。

The embedded software related to this method is located in the `TriangularDitherMethod `directory.


# Comparing the first and second methods(比较两种方式)
基于使用白噪声的过采样和平均的方法，过采样率的每加一倍将提升半位额外分辨率。 最大输入频率随着需要提升的位数的增加而急剧降低。

对于此增益足够的应用，这是一个不错的选择。 它需要输入信号中存在白噪声才能使信号转换结果在两个相邻的 ADC 输出值之间切换。 一般来说，ADC热噪声就足够了，不需要添加外部硬件来充当外部白噪声源。 这使得该解决方案更具成本效益。

第二种方法基于使用三角波形抖动输入信号并计算其在两个量化步骤之间的相对位置，过采样率的每增加一倍可以提升一位的分辨率。 这是第一种方法给出的改进的两倍。要使该方法起作用，输入信号不得与三角信号相关，并且在过采样期间不得有大于 0.5LSB 的变化。 然而，需要外部硬件来实现在输入信号上添加一个三角波。

表 1 总结了两种方法之间的主要区别。 不能说一种方法比另一种方法好。 每种方法都有其优点和局限性。 用户必须选择更能满足其应用要求（采样频率、有效位数等）的一种。

![](Pasted%20image%2020230319182555.png)

---------------

# Hints(建议)

## What is the maximum number of bits that can be added to the on-chip ADC resolution?(片上ADC能增加的最大分辨率位数是多少？)
可以很容易地证明，增加片上 ADC 分辨率会降低输入信号的最大频率分量。
例如，当在 1 MHz 下使用 STM32F1 系列、STM32F3 系列或 STM32Lx 系列 ADC 并且应用需要增加2位分辨率时，则最大输入频率除以：
- 16 当使用白噪声方式时。(62.5 kHz)
- 4 当使用三角抖动方式时。(125 kHz)

**可以增加的最大有效位数？**
对于这两种方法，输入信号的估计是在 OSR 乘以 ADC 转换速率的过采样周期内完成的。 在这种情况下，ADC 以 1 MHz 运行，输入信号估计在 OSR µs 内完成。 对于白噪声方法，信号变化不得超过 1/2LSB，对于三角波方法，信号变化不得超过 ±0.5LSB。
- 使用白噪声方法时，可添加到 ADC 分辨率的最大位数仅取决于输入信号。
- 使用三角抖动方法时，可添加到 ADC 分辨率的最大位数不仅仅取决于输入信号。 事实上，定义三角波信号的步骤取决于 ADC 和 APB 频率。 定时器周期应等于 ADC 速率：
	![](Pasted%20image%2020230319183511.png)

在我们的示例中，以 1 µs 的速率运行 ADC 会导致 STM32F1 系列以 56 MHz 的频率运行，这意味着定时器周期必须等于 55。在这种情况下可以添加的最大位数是 4。


## Taking advantage of STM32 DAC implementation
一些 STM32F1 系列、STM32F3 系列和 STM32Lx 系列设备带有 DAC（数模转换器），可用于过采样方法以避免使用外部组件。
DAC可用于以下两种过采样方式:
- 在第一种方法中，DAC 可用于生成幅度可编程的白噪声波形，如果噪声不足，可将其注入输入信号。 由于实施了伪随机算法，因此生成了波形。 更多详细信息，请参阅 STM32F1 系列、STM32F3 系列和 STM32Lx 系列参考手册。
- 在第二种方法中，DAC 可用于生成三角波。 这样就无需任何额外的外部 RC 电路来过滤定时器 PWM 频率。
_Note: 这部分内容未在应用说明中所给出的软件中实现。_

## Taking advantage of the STM32F1 Series, STM32F3 Series and STM32L4 Series dual ADC mode implementation
在一些STM32F1系列、STM32F3系列和STM32L4系列设备中，双ADC模式是一个有趣的特性，允许两个ADC同时进行转换。

使用双ADC快速交错模式，同一通道由ADC2和ADC1交替转换。 分隔两个连续样本的时间是 7 个 ADC 时钟周期。 因此，输入信号的过采样速度更快。 在本应用笔记描述的示例中，每 1 µs 获取一个样本。 使用双 ADC 快速交错模式，可以每 7 个 ADC 时钟周期进行一次采样，即在 14 MHz 下运行 ADC 时每 0.5 µs。
_Note: 这部分内容未在应用说明中所给出的软件中实现。_

## Taking advantage of the STM32L0 Series hardware ADC oversampling implementation
在 STM32L0 系列和 STM32L4 系列设备上，ADC 在硬件中实现了过采样功能。

在硬件过采样 ADC 模式下，过采样（平均）由硬件计算最终结果，并提高分辨率。 ADC 在内部执行预定义数量的数据转换，这些转换平均为一个标准的最终 ADC 结果。 从而减少CPU或者软件干预，降低微控制器消耗和程序内存使用。 最终的过采样样本与软件过采样方法得到的样本相同。
_Note: 有关硬件和软件过采样使用情况的比较，请参阅“STM32L0 系列和 STM32L4 系列微控制器的 ADC 硬件过采样”应用说明 (AN4629)。_




------------------------------------------------
# 附录A ：量化误差

假设用户有一个 N-bit 的 ADC 和 一个参考电压 V<sub>AREF</sub>。
量化区间 q 在两个相邻的ADC值之间，如下定义：
$$ q = \frac{V_{AREF}}{2^N}  $$
量化误差等于：(即最大的误差可以达到0.5LSB)
$$ |e_q | \le \frac{q}{2} $$
假设：
- 信号样本跨域了多个电平级数。
- 采样频率与信号信号频率不同步。
- 输入信号在量化区间q中的任意位置概率相同，导致随机量化误差。

根据以上的假设，量化噪声可以近似为平均值为0的在ADC分辨率之间均匀分布的随机变量。根据这个假设，可以很容易地证明 **量化噪声方差** 由以下公式给出：
(没看懂，只知道最终量化噪声方差为 q平方除12， 那么提升ADC分辨率，即可减小量化噪声方差)
![](Pasted%20image%2020230319144740.png)
根据以上公式，**量化噪声功率会因为ADC分辨率上升而急剧降低**。

给定一个ADC 采样频率 f<sub>S</sub>(实际根据MCU而异)，根据香农定理，**量化噪声功率密度PSD**为：
![](Pasted%20image%2020230319145406.png)
假定f<sub>m</sub>是输入信号的最大频率分量， 在**有效频段中存在的量化噪声功率**由以下公式给出：
![](Pasted%20image%2020230319145730.png)
![](Pasted%20image%2020230319145929.png)
注意，增大采样频率可以降低频带内的量化噪声功率，从而提高信噪比。

给定一个输入信号，并使用 2.f<sub>m</sub> 和 f<sub>S</sub> = 2.N.f<sub>m</sub> 进行采样，信噪比增益为：
![](Pasted%20image%2020230319150301.png)
总结一下：暂时没看懂，大概是说可以通过提升ADC分辨率或者采样频率提高信噪比。



# 可以帮助理解的一些内容：
https://blog.csdn.net/weixin_38345163/article/details/129355715
https://zhuanlan.zhihu.com/p/350933559
https://zhuanlan.zhihu.com/p/364669403