# unity 内置渲染管线 效果调整工作流程

本次工作流程使用的是unity 2021.3.0f1，主要针对表现效果较差的webgl的效果调整，同样适用于pc端的效果调整，只是在shader的表现上有所差异。（不涉及烘焙内容）

以下的工作流程只是个人的经验总结，欢迎指导。

**以下图场景为例**

![](https://s2.loli.net/2022/08/12/hpnTYjLOvSZEU8V.png)

### 第一步.直接光照调整

- Directional Light（平行光）

  - 决定了漫反射光照强度
  - 阴影方向
  - 阴影强度    

  调整说明：

  ![](https://s2.loli.net/2022/08/16/3gEWiU4mxtRyBNe.png)

  **Intensity**：光照强度（越高直接光越亮）

  **Strength**：阴影强度

  shadow Type（阴影类型）

  - soft shadows——阴影边缘比较柔和，但是漏光明显
  - hard shadows——阴影边缘比较硬

  ![](https://s2.loli.net/2022/08/12/F6zkfitl8NePn2v.png)



### 2.全局光照（环境光）

Window→Rendering→Lighting→Environment

- Environment Lighting（光源）

  - Source
    - Skybox - 采用天空球的光线作为环境光源
    - Gradient - 自定义环境光的颜色（推荐使用）

- Environment  Reflections（反射）

  - Source
    - Skybox  - 采用天空球作为反射参数
      - Resolution 参数的尺寸
    - Custom - 采用自定义天空盒作为反射参数  

  **调整说明：**

  **1.Skybox模式：**

  Environment Lighting和Environment  Reflections的Source都选Skybox

  ![](https://s2.loli.net/2022/08/12/O9eAiBQTqN8LMap.png)

  intensity Multiplier 代表光照的强度

  Environment Lighting（环境光源）的强度对Metallic（金属度）越低的材质影响越高

  Environment  Reflections（反射）的强度是整体影响的。

  ![](https://s2.loli.net/2022/08/12/niyaA7SrIeDxq3N.png)

  ------

  **2. Gradient模式：**

  Environment Lighting的Source选Gradient；

  Environment  Reflections的Source选Coustiom；

  ![](https://s2.loli.net/2022/08/12/5Nw1gLmxOojXkuI.png)

  Environment Lighting分别对color进行调整

  Environment  Reflections的Intensity Multiplier对强度进行调整

  这里的CubeMap用了一张渐变贴图，需要把贴图的Inspector中把TextureShape需改成Cube，贴图可以是彩色的这里主要是天空，反射的光线会加上Cubemap贴图的颜色。

  ![](https://s2.loli.net/2022/08/12/t1SJuirKq9HBocG.png)

  后面的示例延用Gradient模式的效果

### 3.渲染模式

内置渲染管线在Main Camera→Camera→Rendering

![](https://s2.loli.net/2022/08/12/jgb64nR9VYfTPXi.png)

| 渲染模式             | 抗锯齿效果 | 支持空间映射 |
| -------------------- | ---------- | ------------ |
| Forward（前向渲染）  | 好         | 不支持       |
| Deferred（延迟渲染） | 不好       | 支持         |

为了后面的反射效果，牺牲了抗锯齿用了延迟渲染。

### 4.抗锯齿

使用unity官方提供的PostProcessing。Window→PackagesManager

![](https://s2.loli.net/2022/08/12/25zR1PgQJ4coZND.png)

注意，要在Unity Registy的模式下搜索，安装后右下角会是Remove，否则是Install。

在Main Camera 添加Post-process Layer组件，Main Camera 所在的Layer必须和Post-process Layer里的Layer一致。

![](https://s2.loli.net/2022/08/12/U6ndkEB1elOHKMq.png)

 Anti-aliasing Mode

- MSAA像素加点采用
- FXAA替换边界
- TAA复用上一帧（推荐）  

如果觉得效果不明显还可以叠加抗锯齿的脚本，下面Effect会提到

![](https://s2.loli.net/2022/08/12/t1SJuirKq9HBocG.png)

![](https://s2.loli.net/2022/08/12/yfovKGt8aUSqegJ.png)

抗锯齿效果对比

### 5.Effect

这里没有继续使用PostProcessing，原因是它对webGL的支持不太好，所以使用了很久以前的Effects脚本工具包，效果是一样的。

#### Tonemapping  色调映射		

![](https://s2.loli.net/2022/08/12/stXwP3o9SigcnEN.png)

这里选择Neutral 可以调整的参数多一点

- Exposure（曝光度）整体调整场景的明暗  

- 下面的参数都可以调整，对场景的明暗都会有影响

  ![](https://s2.loli.net/2022/08/12/ypwVfIWXxPvLh3N.png)



#### Color Grading  颜色分级

![](https://s2.loli.net/2022/08/12/dOuEZ2v3lcnIbhf.png)

- Shadows、Midtones、Highlights（阴影，中间色，高光的色相）

- Temperature Shift（色温） ——冷暖色调

- Tint（色调）——一般不动

- Hue（色相）——和色调一样不动

- Saturation（饱和度）——越高颜色越鲜艳，默认1，可以稍微调大一点

- Vibrance（自然饱和度）——越高颜色越鲜艳，默认0

- Value（明暗度）——默认1

- Contrast（对比度）——越高画面越明显，默认1，可以稍微调大一点

- Gain（增益）——默认1，轻易不要动

- Gamma（伽马校正）——默认1，轻易不动

  ![](https://s2.loli.net/2022/08/12/jhoeFt95NO1YGwS.png)

  

#### Screen Space Reflections 屏幕空间反射

必须满足上文提到的渲染模式

![](https://s2.loli.net/2022/08/12/L2CByN9ek4KIhmZ.png)

- Max Distance （最大映射距离）

- Iteration Conut(迭代数量)

- Step Size（步长）——太高会出现摩尔纹

- Width Modifier(宽度修饰)——太低和影响映射的完整度

- Reflections Blur（反射模糊）——越低越清晰

- Reflections  Multiplier（反射乘数）——影响反射的颜色，越低越黑

- Fade Distance （消失距离）  

  ![](https://s2.loli.net/2022/08/12/clAQONCIVJMyES2.png)

  ![](https://s2.loli.net/2022/08/12/AQF5oGX9dPagu6f.png)



#### Ambient Occlusion 环境光遮蔽

![](https://s2.loli.net/2022/08/12/cSV1I53QvL79qmo.png)

- Intensity（强度）
- Radius（半径）  

这里的值都不要太高，画面容易脏

效果对比

使用前

![](https://s2.loli.net/2022/08/12/PSuQCq9TBRO248y.png)

使用后

![](https://s2.loli.net/2022/08/12/NMDLAZsjWz5mbOx.png)

（该模型在建立时，就进行过贴图的AO烘焙，效果不是很明显）



#### Depth of Field 景深

![](https://s2.loli.net/2022/08/12/8bhNvTRFSjV4XaI.png)

- Quality——选择只影响远处
- Focal Piont（焦点）
- Smoothness（平滑度）  

效果对比

使用前

​	![](https://s2.loli.net/2022/08/12/akvLfjZoiUmT8hE.png)	

使用后

![](https://s2.loli.net/2022/08/12/AJNS5BDOZ6pHePE.png)



#### Bloom 辉光

![](https://s2.loli.net/2022/08/12/eDMXKVZ3zChaSjE.png)

- Threshold （阈值）——越高，辉光越明显
- Soft knee（曲线缓变弯折处）——越高，影响到的色彩范围越大，使用时不要拉到1，留出高光位置
- Radius（影响半径）
- Intensity（强度）
- Dirt Texture（辉光图）——可以赋值一只图片，可以起到类似滤镜的作用
- Dirt  Intensity（辉光图的强度）    

效果对比

使用前

![](https://s2.loli.net/2022/08/12/UmhYFdyDIEGuxtp.png)

![](https://s2.loli.net/2022/08/12/Q5H62NihqPkxYvF.png)



### 6.天空球

使用天空球，要回到第二步全局光照（环境光），修改成Skybox模式，后面的调整都是基于全局光照和直接光照固定的情况下。因为全局光照的变化对整个场景的色调影响很大，因此室内和室外的效果调整是完成不一样的，unity的 HDRP 渲染设置引入了Volume的方式重新定义了全局和局部的效果调整，后面会更新unity HDRP 渲染设置。

#### **天空球的模式**

天空球有两种模式，6sided和cubemap

**6sided**

是由六张贴图组成的盒子

![](https://s2.loli.net/2022/08/15/ptZrYoxA1V5TUgD.png)

建立的效果

![](https://s2.loli.net/2022/08/15/BIzAUire8CkTmgc.png)

**cubemap**

是由一张张贴图组成的盒子

![](https://s2.loli.net/2022/08/15/SMwhrW4K9Og1JpQ.png)

这里有几点要注意：使用贴图做天空球，要用高清的图片，导入后也要设置在1024以上；相较起来6张的效果会比一张的好一点。

#### 加入天空盒子球的效果

![](https://s2.loli.net/2022/08/15/QTtdcvLw9ABEgJy.png)

### 7.材质

这里以unity 内置渲染管线的Standard为例

![](https://s2.loli.net/2022/08/16/P7fKNYMcWJb5XZV.png)

#### Metallic 金属度

0-1代表金属度 ，金属度越低，环境光对材质的影响越大。

![](https://s2.loli.net/2022/08/16/9oiZA2VsnuqKtOv.png)

![](https://s2.loli.net/2022/08/16/N35FY6sbnJPekxc.png)

再开始调整前，可以适量的将金属度调低，方便颜色调整后进行微调。因为天空的亮度比较高，如果金属度太高，后面整体调整时会出现要么天空曝光，要么模型太暗。

#### Smoothness 光滑度

0-1代表光滑度，值越大，菲涅耳反射效应越明显。

*Smoothness 为0*

![](https://s2.loli.net/2022/08/16/r6SYOBKej5Llmoy.png)

*Smoothness 为1*

![](https://s2.loli.net/2022/08/16/QaeLGtDwY6zR1Xp.png)

#### Emission自发光

配合Bloom 可以实现高光的效果；也是提高材质亮度和颜色的手段

#### Transparent 透明

渲染模型改为Transparent  ，Albedo颜色的alpha值调低。

透明效果

![](https://s2.loli.net/2022/08/16/7nCMmBFWHwuT3Q5.png)

不透明效果

![](https://s2.loli.net/2022/08/16/t5xiaRQGTwJfhL7.png)

#### 最后的效果

![](https://s2.loli.net/2022/08/16/i5HXJRdA89VszQ1.png)

![](https://s2.loli.net/2022/08/16/utaCEMkVsp3IfRb.png)

![](https://s2.loli.net/2022/08/16/GWVhfPYA3suHpUr.png)
