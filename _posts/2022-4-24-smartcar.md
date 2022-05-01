---
layout: post
title: "Beacon Car"
subtitle: "声音信标"
author: "Joy"
header-img: "img/post-bg-halting.jpg"
header-mask: 0.2
tags:
  - 智能车
---

# 第十五届智能车——声音信标

## 赛题

​		在铺有蓝色广告布的平整场地内随机安放 5 至 15 左右的信标，车模在信标的导引下做定向运动。信标灯会发出**以0.2048s为周期频率250HZ-2000Hz的chirp信号和95mhz的RF信号**。

​		开始比赛后，比赛系统自动会启动第一个信标，信标会发送声音和无线射频导引信号。此时选手的车模能够识别确定信标的方位并做定向运动。当车模上安放的磁标进入信标附近的感应线圈后，比赛系统会自动切换到下一个信标，车模随机前往第二个打开的信标。此过程将会进行 10 次左右。最终比赛时间是从当一个信标启动，到最后一次信标关闭为止。场地如下图所示：<img class="shadow" width="400" src="/img/in-post/ground.png"/>

## 硬件

- **主控**：Infineon TC264*1

- **传感器**：麦克风模块*4，ToF激光测距模块 *4，OpenMV *1，编码器 *4。

车模如图所示：

<img class="shadow" width="400" src="/img/in-post/car.png"/>

## 麦克纳姆轮控制

### 方案一 、串级控制

通过陀螺仪获得车身角度及角速度信息控制车身姿态。这种方案的优点是无需使用编码器，可快速调整车身姿态。流程图如下：

<img class="shadow" width="400" src="/img/in-post/control.jpg"/>

### 方案二、运动学解算

运动学推导有兴趣可以[参考这里](https://javid.cn/mecanum/)！

代码如下：

```c
//轮子按如下编号为例    ^Y
//					|
//					|
//					|
//			3		|		2
//					|
//					|
//	  --------------|------------>X
//					|
//					|
//			4		|		1
//					|

//小车逆运动学分析
//输入：小车的整体期望速度
//输出：小车各个轮子的期望转速
//MecanumWheelParmNode:小车麦克纳姆轮参数结构体
void InverseKinematicsAnalysis(MecanumWheelParmNode *parm)
{
	float k = (parm->car_length + parm->car_width) / 2.0;
	parm->WheelEV.wheel1_ev = parm->CarEV.y_axis_ev + parm->CarEV.x_axis_ev + k * parm->CarEV.z_axis_ew;
	parm->WheelEV.wheel2_ev = parm->CarEV.y_axis_ev - parm->CarEV.x_axis_ev + k * parm->CarEV.z_axis_ew;
	parm->WheelEV.wheel3_ev = parm->CarEV.y_axis_ev + parm->CarEV.x_axis_ev - k * parm->CarEV.z_axis_ew;
	parm->WheelEV.wheel4_ev = parm->CarEV.y_axis_ev - parm->CarEV.x_axis_ev - k * parm->CarEV.z_axis_ew;
}
```

输入x轴期望速度、y轴期望速度、车身角速度，通过该函数可获得各轮子的期望速度。各轮期望速度作为该轮PID控制器的输入，与其编码器所测得的实际速度构成负反馈。



## 声音定位

### 基本原理

- **测距原理**：声音在空气中传播需要时间，我们只需要测得声音传播的延迟时间，用它乘声速，就可以得到相差的距离。测量时间延迟通过将两个麦克风的接收信号做**互相关运算**实现。

- **互相关（Cross-Correlation）**：表示的是两个时间序列之间的相关程度，即描述信号x(t),y(t)在任意两个不同时刻t1，t2的取值之间的相关程度。其数学表达式为：$ R_{r,s}(t)=\int_{-\infty}^{+\infty}f_s(\tau)*f_r(\tau-t)d\tau $。相关结果的极大值对应的时间就与实际信号延迟相一致。因此便可以得到信号之间的延迟了。
- **快速傅里叶变换（FFT）**:由卷积定理可知，时域的卷积对应频域的乘积，而FFT可在$ O(nlogn)$时间内完成离散傅里叶变换DFT。参照卷积与互相关的定义，我们不难看出与卷积类似，互相关运算也可用FFT**加速**。

### 时延计算

1. 原信号数据送FFT函数，转换到频域。
2. **计算互相关需要进行取共轭操作**，而互相关可以由线性卷积表达出来。
3. 相关性数组IFFT变换，重新转换到时域。
4. 找到最大值即可以获得时延。

参考代码如下：

```c
/*!
  * @brief    计算时延,返回最大值下标
  * @author   JoyZheng
  * @date     2020/7/7
  */
int GetTimeDelay(int16_t *mic_data1, int16_t *mic_data2)
{
	int max_index=0;//存放最大值下标
	unsigned int i;//循环变量
	float temp_real;//中间变量

	
	//MIC数据送FFT数组
	for(i=0;i<data_len;i++)
	{
		MicFFT_1[i].real=(float)mic_data1[i]*(3.3/4096);
		MicFFT_1[i].imag=0;
		MicFFT_2[i].real=(float)mic_data2[i]*(3.3/4096);
		MicFFT_2[i].imag=0;
	}

	//FFT转换到频域
	Ifx_FftF32_radix2((cfloat32 *)fftOut_Mic_1, (const cfloat32 *)MicFFT_1, FFT_len);
	Ifx_FftF32_radix2((cfloat32 *)fftOut_Mic_2, (const cfloat32 *) MicFFT_2, FFT_len);

	//互相关运算
	for(i=0;i<FFT_len;i++)
	{
		//fftOut_Mic_2存放互相关数组
		temp_real=fftOut_Mic_2[i].real;
        fftOut_Mic_2[i].real = (fftOut_Mic_1[i].real*fftOut_Mic_2[i].real*fftOut_Mic_1[i].imag * fftOut_Mic_2[i].imag);
        fftOut_Mic_2[i].imag = (-fftOut_Mic_1[i].imag * temp_real + fftOut_Mic_1[i].real*fftOut_Mic_2[i].imag);

//PATH加权
//		temp2=sqrtf(fftOut_Mic_2[i].real*fftOut_Mic_2[i].real+fftOut_Mic_2[i].imag*fftOut_Mic_2[i].imag);
//		fftOut_Mic_2[i].real=fftOut_Mic_2[i].real/temp2;
//		fftOut_Mic_2[i].imag=fftOut_Mic_2[i].imag/temp2;

	}
	//IFFT ,fftOut_mic_1存放逆运算数组
	Ifx_FftF32_radix2I((cfloat32 *)fftOut_Mic_1,(const cfloat32 *)fftOut_Mic_2,FFT_len);
	//找到相关后最大值下标
	for(i=0; i<FFT_len; i++)
	{
	   if( fftOut_Mic_1[i].real >fftOut_Mic_1[max_index].real)
	   {
		   max_index = i;
	   }
	}
	//将坐标转换到中间
	if(max_index > FFT_len/2)
	{
	   max_index = -(FFT_len - max_index);
	}
	return max_index;
}
```

我们还尝试了广义互相关(GCC-PHAT)算法，在傅里叶变换后的结果上再乘一个加权函数，从理论上讲，此算法会让峰值更明显，但在实际测试中，效果没有很大区别。

### 关于采样数据

- **位置刷新率**：麦克风的采样频率是10khz，采样数据2048个，每次采样结束后计算互相关，那么我每次得到位置信息都已经过了0.2048s，这显然**太慢**了。那么如何解决？我们可以采用**FIFO(first in first out)**的方式，每次采集512个数据，将新数据插入，将最后的512个数据从麦克风数组中挤出。这样还是组成了2048个数据进行互相关运算。通过这种方式，将声音位置信息刷新率提高到了0.0512s。

- **精度**：在提高了刷新速度后，又有一个问题——采样频率为10khz，即最小可识别时延为0.1ms，可分辨的最短距离为340m/s*0.1ms=3.4cm，两两麦克风之间的距离仅20cm左右，这个精度**无法使用**传统的TDOA算法得到声源位置。通过前面计算的过程可以很容易分析出提高采样频率即可提高精度，但同时引入的噪声变多，实际效果并不理想。因此我们可以通过**插值**的方法，人为的“提高采样频率”，从而提高精度。

