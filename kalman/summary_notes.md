# 卡尔曼滤波器在姿态估计中的应用的简略总结
无数的人耗费了无数的墨水和口水。我也假装我懂了。-- 题记

* * *

# 前言
从数学到物理，从理论到工程，从物理世界的模型到测量噪声假设，从姿态估计的原理到卡尔曼滤波器的设计，从卡尔曼滤波器的原理到验证，迷魂阵一样，还缺一不可。耗时半年，就是为了看懂几百行代码，太难为人了。比起那些月产出万行代码的高手，实在是惭愧。

也许那些大神说得对，我们就是这样认识世界的：睁眼瞧瞧这个世界，再闭眼猜猜这个世界，然后按照自己的感觉调整调整 Q & R 矩阵，如此循环往复。只闭眼瞎猜的，那是傻子，只睁眼盯着的，那是呆子。能把 Q & R 矩阵调整得既能猜得准，又能灵敏反应变化的，那就是大神了。

从某种哲学意义上来说，这就是我们的现状：人类自以为聪明，掌握了一些这个世界的运行规律，称之为科学原理，又掌握了一些感知这个世界的能力(make sense)，称之为测量方法。其实两者都有问题，总结规律有谬误，测量数据有误差。于是大神找到了应付现状的最佳方法：
* 闭着眼睛用瞎凑的公式猜测这个世界从上一个状态运行到新的状态，这就是预测(Predict)，这是这段时间内这世界按照某种规律运行的变化结果，所以又叫时间更新(Time Update)；
* 公式有问题，于是就睁开眼睛看看有多大的差异，这就是修正(Correct)，这是测量得到的世界的变化，所以又叫测量更新(Measurement Update)；
* 两者都有误差。于是假设误差按照正态分布，再按照最小二乘法进行优化，最佳估计值是公式值和测量值各乘上各自的信任系数后相加而来，这就是卡尔曼滤波器。

# 姿态估计的基本原理
姿态估计可以采用很多种方法。卡尔曼滤波器也有很多应用领域。这两者并不需要绑定在一起才能工作。但自从在阿波罗登月计划中成功应用开始，采用卡尔曼滤波器进行姿态估计，就是最佳的工作模式，同时也一再证明了卡尔曼滤波器的神奇。

## 重力分解法
根据中学所学的重力分解法，很容易理解用重力矢量估计机体姿态的原理。根据重力在 3D 坐标轴上的分量，解三角函数就可以求出当前机体的姿态。

当然，因为是 3D 空间，计算相对复杂，再考虑实际环境情况，处理起来有些麻烦。为此，制造传感器的厂商总结了不少算法资料：
+ [ST集成传感器方案实现电子罗盘功能](http://ic.semi.org.cn/a/technology/mems/26112.html)
+ [ST AN4509 - Tilt measurement using a low-g 3-axis accelerometer](https://www.st.com/resource/en/application_note/dm00119046-tilt-measurement-using-a-lowg-3axis-accelerometer-stmicroelectronics.pdf)
+ [NXP AN4248 - Implementing a Tilt-Compensated eCompass using Accelerometer and Magnetometer Sensors](https://www.nxp.com/assets/documents/data/en/application-notes/AN4248.pdf)
+ [AN005 - Tilt-Sensing with Kionix MEMS Accelerometers](http://kionixfs.kionix.com/en/document/AN005-Tilt-Sensing-with-Kionix-MEMS-Accelerometers.pdf)
+ [AN006 - Handheld Electronic Compass Applications Using a Kionix MEMS Tri-Axis Accelerometer](http://kionixfs.kionix.com/en/document/AN006%20Handheld%20Electronic%20Compass%20Applications%20Using%20a%20Kionix%20MEMS%20Tri-Axis%20Accelerometer.pdf)

这种方法有一个优点，就是因为数据从 IMU 中读出，而惯性是机体的内在禀性，所以重力测量相对稳定，很难被干扰。如果机体旋转到在任意位置测量重力矢量，综合得到的是一个圆球。

用以上总结出的算法，可以方便快捷地算出机体的姿态。但是，加速度计测量得到的是重力和线性加速度的叠加。这两者在数值中无法区分。于是这种方法只适用于静态机体或转动很慢的机体。

## 磁力分解法
和分解重力矢量的原理类似，根据测量到的地磁矢量，解三角函数也可以求出当前机体的姿态。

但是地磁测量得到的不是一个圆球，而是一个椭球。而且地磁很弱，不容易测量。因此地磁很容易受到环境的干扰，很不可靠。

地磁测量值也因此很难校准。都是保密算法，或者是有专利的算法。下面是 NXP 所使用校准的算法及软件架构：
+ [NXP AN5019 - Magnetic Calibration Algorithms](https://www.nxp.com/docs/en/application-note/AN5019.pdf)
+ [NXP® E-Compass Software](https://www.nxp.com/design/development-boards/freedom-development-boards/sensors/nxp-e-compass-software:E-Compass)

因为数据很不可靠，所以使用地磁矢量一般都是最后的方法。

## 角度积分法
前两个是绝对法，确定机体姿态和上一次测量无关，只和本次测量有关。

角度积分法是相对法，确定机体姿态和上一次姿态有关，和本次测量有关，是两者的叠加。实际上就是确定初始状态后，到现在的所有测量结果的积分。

从 IMU 中读出的是角速度数据。角速度乘以采样间隔时间就得到了这段时间内机体姿态角度的变化，再通过算法叠加到机体的当前姿态，就得到了新时刻的机体姿态。因为角速度数据从 IMU 中读出，而惯性是机体的内在禀性，所以读出的角速度就很精确，不会被外界干扰。所以这是机体姿态估计中最根本的算法。大多数姿态估计的解题思路都从这个公式起步。

因为角速度数据精确，所以角度积分法短期可信，但是该方法长期不可信，因为积分有积累误差，时间越久误差越大。重力法和磁力法短期不可信，长期可信。这是因为采用的是绝对法，基于统计学，基于零均值高斯白噪声假设，该估计方法的误差会稳定在一定的数值上。

# 使用卡尔曼滤波器的姿态估计的解题思路

对于机体坐标系来说，姿态的变化量，可以从角速度推算而来。于是角速度对于 KF 系统，就是系统驱动力。绝大多数姿态估计的系统，都是由角速度进行驱动。重力数据和地磁数据用于校正。卡尔曼滤波器用于姿态估计，一个通用的离散算法流程大约是：
1. 从陀螺仪获得角速度，应用姿态变化的微分方程(动力学方程)，积分得到当前姿态。这一步，在 KF 里就是时间传播，就是预估步骤。计算得到的就是姿态的估计值，短期有效，长时间有较大累积误差，所以需要校正。
2. 从加速计获得包含了重力的加速度，对姿态的估计值做校正。
3. 从磁力计获得地磁数据，可以对姿态的估计值做校正。
4. 从其它和姿态有关的传感器获得数据，对姿态的估计值做校正。
5. 回到步骤1，周而复始地估计物体的姿态。

用重力数据和地磁数据校正，最常用的方法就是固定矢量观测。从重力矢量计算，可以得到以重力矢量为法向量的当地水平面，从而校正了欧拉角中的 Roll & Pitch。结合已知的水平面，从地磁矢量分离出水平面上指向磁极的矢量，从而校正了欧拉角中的 Yaw。

把重力矢量做为一个固定矢量观测，原理是这样的：当地水平面的法向量是重力矢量。我们已经猜测机体坐标系已经旋转一个角度，我们反方向旋转回来，机体坐标系的 Z 轴应该与观测到的重力矢量重合。如果有差异，就要相互校正。

但是，从加速计获得的是叠加了重力矢量的加速度矢量。所以这里有一个重要假设，就是把外力当成是零均值高斯白噪声的一部分，短期内各个方向各种大小的外力随机分布，均值为零。于是这种算法最怕机体长时间只受一个恒定的力的情况，例如自由落体、或抛物线运动。

在微重力环境，例如在太空船中，使用星光或太阳传感器做为参考固定矢量。这比观测重力矢量更准确。但是成本不知几何，不是民用设备能用得起的。

把地磁矢量做为一个固定矢量观测，原理和上面类似：地磁矢量分离出水平面上的矢量指向磁极。我们反方向旋转回来，机体坐标系的 Y 轴应该与观测到的地磁水平矢量重合。如果有差异，就要相互校正。这种方法使用的是地磁矢量在水平面上的分量。在高维度地区，地磁矢量差不多指向地心，在水平面上的分量很小。因此该算法在高维度地区会不可用。此外，地磁很弱，不容易测量，测量得到的不是一个圆球，而是一个椭球。同时地磁容易受到环境的干扰，不可靠。所以，如果有其它方法得到 Yaw 值，应该优先采用其它方法。参考地磁矢量是最后的方法。

# 用于姿态估计的卡尔曼滤波器的选择

应用于姿态估计的卡尔曼滤波器的滤波器种类有很多，每年的论文层出不穷，每隔几年就会有新的种类冒出来，各个都自称最优最精确。那该选哪一个进行学习呢？

其实有一个简单的办法可以选择，那就是：看航天人的。

自从卡尔曼滤波的第一个应用，也是第一个成功的应用---阿波罗登月计划---起，航天人一直都是卡尔曼滤波研究的领军人物。别人可以灌水论文提职称，而航天人需要对运行十余年，耗资十几亿美元的项目负责。灌水一时爽，秋后算账惨，航天人是不敢的。

在工业届里，非线性系统的估计最常使用的就是 EKF 与 UKF。在姿态估计的方案里，也经常有这两种的选择。那么在实际项目里，EKF 与 UKF，该选择哪一个呢？搜索下来，主要还是选择了 EKF。理由大概有这几个：
* 根据不少论文里的实测，EKF 与 UKF 得到的估计值的精度，也就是最小均方误差(MMSE)，基本上差不多。也许是姿态估计中的非线性方程中的线性性质较多，EKF 已经足够应付。也许是低成本的传感器的测量噪声太大，体现不出 UKF 的优点。总之，EKF 在估计精度上没有明显劣势。
* 根据复杂度分析，UKF 大概是 EKF 的两倍。这在嵌入式系统中是很敏感的指标。EKF 因为计算量较小，速度更快，能够适应于 MCU 系统。所以选择 EKF 的就比较多。

在 EKF 中，针对姿态估计的特点，大多又选择了 ESKF。之所以 ESKF 会占优势，是因为它在数学概念上更合理。在这里，[Dr. F. Landis Markley](http://www.acsu.buffalo.edu/~johnc/markley/) 这位白胡子老爷爷值得提一提，就是他从数学上证明了 ESKF 在数学概念上更合理，至少到二阶项都是符合 SO3 和卡尔曼滤波的约束的。于是 ESKF 就成了主流。

所以，本地的大多数资料都是针对 ESKF 应用的分析。

## 扩展卡尔曼滤波器中角速度的应用

AEKF(Additive Extended Kalman Filter)是最先设计出来的用于姿态估计的算法。最初始的来源，还是角度积分法和四元数微分的概念。

根据四元数微分的定义：

<a href="https://www.codecogs.com/eqnedit.php?latex=\dot{q}(t)=\lim\limits&space;_{\Delta&space;t\to0}\dfrac{q(t&plus;\Delta&space;t)-q(t)}{\Delta&space;t}" target="_blank"><img src="https://latex.codecogs.com/svg.latex?\dot{q}(t)=\lim\limits&space;_{\Delta&space;t\to0}\dfrac{q(t&plus;\Delta&space;t)-q(t)}{\Delta&space;t}" title="\dot{q}(t)=\lim\limits _{\Delta t\to0}\dfrac{q(t+\Delta t)-q(t)}{\Delta t}" /></a>

推导出微分四元数和角速度的关系：

<a href="https://www.codecogs.com/eqnedit.php?latex=\dot{q}=\dfrac{1}{2}q\omega'" target="_blank"><img src="https://latex.codecogs.com/svg.latex?\dot{q}=\dfrac{1}{2}q\omega'" title="\dot{q}=\dfrac{1}{2}q\omega'" /></a>

其中<a href="https://www.codecogs.com/eqnedit.php?latex=\omega'" target="_blank"><img src="https://latex.codecogs.com/svg.latex?\omega'" title="\omega'" /></a>是在 IMU 中测量得到的角速度矢量。在这里，角速度矢量被简单扩展为实部为'0'的纯四元数。

在根据四元数微分的定义计算积分：

<a href="https://www.codecogs.com/eqnedit.php?latex=\inline&space;\\\dfrac{q(t&plus;\Delta&space;t)-q(t)}{\Delta&space;t}=\Delta&space;q&space;\\q(t&plus;\Delta&space;t)-q(t)=\Delta&space;q\Delta&space;t&space;\\q(t&plus;\Delta&space;t)=\Delta&space;q\Delta&space;t&plus;q(t)=\dfrac{1}{2}q(t)\omega'\Delta&space;t&plus;q(t)" target="_blank"><img src="https://latex.codecogs.com/svg.latex?\inline&space;\\\dfrac{q(t&plus;\Delta&space;t)-q(t)}{\Delta&space;t}=\Delta&space;q&space;\\q(t&plus;\Delta&space;t)-q(t)=\Delta&space;q\Delta&space;t&space;\\q(t&plus;\Delta&space;t)=\Delta&space;q\Delta&space;t&plus;q(t)=\dfrac{1}{2}q(t)\omega'\Delta&space;t&plus;q(t)" title="\\\dfrac{q(t+\Delta t)-q(t)}{\Delta t}=\Delta q \\q(t+\Delta t)-q(t)=\Delta q\Delta t \\q(t+\Delta t)=\Delta q\Delta t+q(t)=\dfrac{1}{2}q(t)\omega'\Delta t+q(t)" /></a>

所以 t + ∆t 时刻的新姿态 q(t + ∆t) 近似为 t 时刻的旧姿态 q(t) 与姿态变化量 ∆q∆t 的叠加。也就是和增量角度矢量(∆θ = ω × ∆t)是线性关系。

但是，上述的微分和积分方程是基于一般四元数的运算进行推导。应用在旋转时没有考虑一个约束：

<a href="https://www.codecogs.com/eqnedit.php?latex=\inline&space;q_{0}^{2}(t)&plus;q_{1}^{2}(t)&plus;q_{2}^{2}(t)&plus;q_{3}^{2}(t)=1" target="_blank"><img src="https://latex.codecogs.com/svg.latex?\inline&space;q_{0}^{2}(t)&plus;q_{1}^{2}(t)&plus;q_{2}^{2}(t)&plus;q_{3}^{2}(t)=1" title="q_{0}^{2}(t)+q_{1}^{2}(t)+q_{2}^{2}(t)+q_{3}^{2}(t)=1" /></a>

也就是只有单位四元数才能表示旋转。运算前后的四元数 q(t) 的范数要一直保持为单位范数。而上述推导中的 '+/-' 运算打破了这个约束。用数学的话来说：代表旋转的单位四元数是群，只有乘法是封闭的。于是上述的角速度矢量和姿态变化的关系在数学上是有问题的。

要解决这个问题，只有请李群和李代数上场。将上述推导中的 '+/-' 运算，改为 '(+)/(-)' 算子，并映射为乘法。同时角速度矢量不再是简单地扩展为纯四元数，在乘以时间间隔得到增量角度矢量之后，要重新计算实部，使得四元数范数保持为'1'。因为采样周期很快，每个时段内的角度变化的数值很小，所以为简化计算，可以简单地将实部写为'1'，计算误差在容许的范围以内。

此外，在卡尔曼滤波器中的核心概念：协方差，其中的 '-' 运算也同样处理。这就是 MEKF(Multiplicative EKF)。此外，角速度矢量如何扩展为代表旋转的单位四元数，还有吉布斯向量(Gibbs Vector)和改进型罗德里格斯参数(Modified Rodrigues Parameters)方式。具体可见参考文档。

以上对于四元数的讨论，同样也适用于旋转矩阵。

## 误差状态卡尔曼滤波器

## 参考帧
从重力矢量，可以得到以重力矢量为法向量的当地水平面，从而校正了欧拉角中的 Roll & Pitch。结合已知的水平面，从地磁矢量分离出水平面上指向磁极的矢量，从而校正了欧拉角中的 Yaw。
重力参考帧的问题。卡尔曼滤波器的假设，零均值高斯白噪声。如果长久受到一个恒定的力，则估计不准确。只受一个力的的自由落体，抛物线运动。
地磁参考帧的问题。地磁矢量的方向，在水平面上的投影。在高维度地区的问题。

在太空船中，使用的星光或太阳传感器做为参考帧。

# 测量值的预处理
## 低通滤波
## 互补滤波
## 地磁测量值的校准

# 地磁测量噪声的隔离

# 特殊应用的特别设计
## NXP的零漂移模型
如何扩展状态列表
## cardboard的预测应用
卡尔曼滤波器的三种应用方向：
1. 平滑
2. 估计
3. 预测

分别代表着对过去、现在和将来时间的事件处理。
1. 无人机对当前姿态的估计和控制，应用的是估计功能。
2. 事后分析，应用的是平滑功能。
3. cardboard 应用的是预测功能。这是因为 VR 应用的特点。

首先，对于显示设备，是按照 VSYNC 信号周期性运行。不能在任意时间更新显示图像。所以用于 VR 头盔的显示屏都要求高刷新率。60Hz 不及格，75Hz 才入门，90Hz 还不差，120Hz 算优秀。

其次，戴着 VR 头盔，从人的头部移动到视网膜上看见相应的图像，有一个延迟时间。在这期间，事件经过传感器采样，状态估计，GPU 根据姿态绘图，然后按照 VSYNC 周期信号显示图像，根据[[Head Tracking for the Oculus Rift](http://msl.cs.uiuc.edu/~lavalle/papers/LavYerKatAnt14.pdf)]一文的研究，这个延迟时间大约是 30 ~ 50ms。

于是，如果按照当前估计姿态绘图，人眼看到的图像表现的是 50ms 前的状态。于是延迟响应就让人的感觉发生问题。

对此的解决方案是根据延迟时间的数值，提前预测未来相应时刻的姿态。然后是根据未来的姿态绘制图像。当人的头部转到位置的时候，人眼也就看到了代表当前姿态的图像。这个延迟时间，在 cardboard 的 JAVA 代码里是硬编码的偏移量，有些写为 33ms(30Hz)，有些写为 58ms。也就是，如果上层代码想预测后面 ∆t 时刻的姿态，实际上预测的是 ∆t + 33 ~ 58ms 时刻的姿态。

应用系统的不同。系统驱动时钟的不同。异步采样，异步到达事件的处理。延迟的处理。

## ecl EKF 的特点
因为应用的特点，必须预测速度。对于无人机，没有速度预测的估计器是没有意义的。加速度的使用方式，用于时间更新而不是测量更新。

# 测试与评价

# 参考资料


