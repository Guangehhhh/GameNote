## RTR_2
---
## 高级着色：BRDF及相关技术

# 球面坐标 SphericalCoordinate 是什么？？
  - 由于光线主要是通过方向来表达，通常用球面坐标表达它们比用笛卡尔坐标系更方便
- 怎么表示球面坐标
  - 球面坐标中的向量用三个元素来指定：
    -   r表示向量的长度
    -   θ表示向量和Z轴的夹角
    -   Φ表示向量在x-y平面上的投影和x轴的逆时针夹角
# 立体角 Solid Angle 是什么？？
  - 是一个物体对特定点的三维空间的角度，是平面角在三维空间中的类比
  - 立体角描述了从原点向一个球面区域张成的视野大小，可以看成是弧度的三维扩展
  - 它描述的是站在某一点的观察者测量到的物体大小的尺度
    - 例如，对于一个特定的观察点，一个在该观察点附近的小物体有可能和一个远处的大物体有着相同的立体角
  - 以观测点为球心，构造一个单位球面
    - 任意物体投影到该单位球面上的投影面积，即为该物体相对于该观测点的立体角
    - 立体角是单位球面上的一块面积，这和“平面角是单位圆上的一段弧长”类似
  - 我们知道弧度是度量二维角度的量，等于角度在单位圆上对应的弧长
    - 单位圆的周长是2π，所以整个圆对应的弧度也是2π
  - 立体角则是度量三维角度的量，用符号Ω表示
    - 单位为立体弧度（也叫球面度，Steradian，简写为sr）
    - 等于立体角在单位球上对应的区域的面积
      - （实际上也就是在任意半径的球上的面积除以半径的平方ω=s/r2 ）
      - 单位球的表面积是4π ，所以整个球面的立体角也是4π

# 投影面积 Foreshortened Area 是什么？？
  - 投影面积描述了一个物体表面的微小区域在某个视线方向上的可见面积
---
# 辐射度学（Radiometry） 是什么？？
- 辐射度学（Radiometry）是度量电磁辐射能量传输的学科，也是基于物理着色模型的基础。
- 为了模拟可见光与各种材质的交互，这个过程涉及到能量的传输

# 辐射度学基本量 都有什么？？
  - 能量（Energy），用符号Q表示，单位焦耳（J）
    - 每个光子都具有一定量的能量，和频率相关，频率越高，能量也越高。
  - 功率（Power），单位瓦特（Watts），或者焦耳／秒（J/s）

# 辐射通量/光通量 Radiant Flux 是什么？？
  - 辐射通量（RadiantFlux，又译作光通量，辐射功率）描述的是在单位时间穿过截面的光能
    - 或每单位时间的辐射能量，通常用Φ来表示，单位是W，瓦特
    - 其中的Q表示辐射能(Radiant energy)，单位是J，焦耳
# 辐射强度/发光强度 Radiant Intensity 是什么？？
  - 对一个点（比如说点光源）来说，辐射强度表示每单位立体角的辐射通量
  - 辐射强度(Radiantintensity，又译作发光强度)，表示每单位立体角的辐射通量
    - 通常用符号I表示，单位 瓦特每球面度
# 辐射率/光亮度 Radiance L 是什么？？

    - 它等于每单位投影面积和单位立体角上的辐射通量，单位是W·sr−1·m−2，瓦特每球面度每平方米
    - 在光学中，光源的辐射率，是描述非点光源时光源单位面积强度的物理量，
  - 一种直观的辐射率的理解方法是：
    - 将辐射率理解为物体表面的微面元所接收的，<<来自于某方向光源>>的单位面积的光通量

    - 辐射率实际上可以看成是我们眼睛看到（或相机拍到）的物体上一点的颜色。
    - 在基于物理着色时，计算表面一点的颜色就是计算它的辐射率
  - 辐射率的微分形式：
    - 其中：Φ是辐射通量，单位瓦特（W）；Ω是立体角，单位球面度（sr）
  - 另外需要注意的是，辐射率使用物体表面沿目标方向上的投影面积，而不是面积。cosA

# 辐照度/辉度 Irradiance E  是什么？？
  - E =Φ /A 表面积
  - 辐照度（Irradiance，又译作辉度，辐射照度，用符号E表示）指入射表面的辐射通量
    - 即单位时间内到达单位面积的辐射通量，或到达单位面积的辐射通量，
    - 也就是辐射通量对于面积的密度，用符号E表示，单位瓦特每平方米。

  - 辐照度可以写成辐射率（Radiance）在入射光所形成的半球上的积分：
    - 其中，Ω是入射光所形成的半球。L(ω)是沿ω方向的光亮度。

# BRDF的定义式  是什么？？
  - 可以将给一个表面着色的过程，理解为给定入射的光线数量和方向，计算出指定方向的出射光亮度（radiance）
  - 在计算机图形学领域，BRDF Bidirectional Reflectance Distribution Function
    - 译作双向反射分布函数，是一个用来描述表面如何反射光线的方程
    - 顾名思义，BRDF就是一个描述光如何从给定的两个方向（入射光方向l和出射方向v）在表面进行反射的函数

  - BRDF的精确定义是出射辐射率的微分（differential outgoing
  radiance）和入射辐照度的微分（differential incoming irradiance）之比：

    - 要理解这个方程的含义，可以想象一个表面被一个来自围绕着角度**l**的微立体角的入射光照亮，而这个光照效果由表面的辉度dE来决定。

  - 表面会反射此入射光到很多不同的方向，在给定的任意出射方向v，
    - 光亮度dLo与辐照度dE成一个比例 而两者之间的这个取决于l和v的比例，就是BRDF。


  - 图6 BRDF图示

  - 一个最常见的疑问是，BRDF为什么要取这样的定义？？
    - BRDF为什么被定义为辐射率（radiance）和辐照度（irradiance）之比
      - 而不是radiance和radiance之比，或者irradiance和irradiance之比呢？
    - 那么，关于这个问题，可以这样理解：
      - 因为照射到入射点的不同方向的光，都可能从指定的反射方向出射
        - 所以当考虑入射时，需要对面积进行积分
        - 而辐照度irradiance正好表示单位时间内到达单位面积的辐射通量
        - 所以BRDF函数，选取入射时的辐照度Irradiance，和出射时的辐射率Radiance
        - 可以简单明了地描述出入射光线经过某个表面反射后如何在各个出射方向上分布
        - 而直观来说，BRDF的值给定了入射方向和出射方向能量的相对量

# BRDF的非微分形式/简单光源  是什么？？
  - f（l,v）=L(v)/E cos角度
    - EL是光源在垂直于光的方向向量L平面测量的辐照度（irradiance）。
    - Lo（v）是在视图矢量v的方向上产生的出射辐射率（radiance）
# BRDF与着色方程  是什么 ？？
  - 非微分的 积分

# BRDF的可视化怎么表示？？
  - 一种理解BRDF的方法就是在输入方向保持恒定的情况下对它进行可视化表示
    - 对于给定方向的入射光来说，图中显示了出射光的能力分布：在交点附近球形部分是漫反射分量
    - 因此出射光来任何方向上的反射概率相等
    - 椭圆部分是一个反射波瓣（ReflectanceLobe）
      - 它形成了镜面分量。显然，这些波瓣位于入射光的反射方向上，波瓣厚度对应反射的模糊性
      - 根据互易原理，可以将这些相同的可视化形成认为是每个不同入射光方向对单个出射方向的贡献量大小
# BRDF的性质  有那些？？
  - 1 可逆性
    - BRDF的可逆性源自于亥姆霍兹光路可逆性（Helmholtz Recoprpcity Rule）
      - BRDF的可逆性即，交换入射光与反射光，并不会改变BRDF的值：

  - 2 能量守恒性质
    - BRDF需要遵循能量守恒定律。能量守恒定律指出：入射光的能量与出射光能量总能量应该相等

  - 3 线性特征
    - 很多时候，材质往往需要多重BRDF计算以实现其反射特性。
      - 表面上某一点的全部反射辐射度可以简单地表示为各BRDF反射辐射度之和。
      - 例如，镜面漫反射即可通过多重BRDF计算加以实现。
# BRDF的模型分为几种 ？？几个类别？？
  - 根据BRDF的定义来直接应用，会有一些无从下手的感觉
    - 而为了方便和高效地使用BRDF数据，大家往往将BRDF组织成为各种参数化的数值模型

  - 有各式各样的BRDF模型，如：
    -   Phong (1975)
    -   Blinn-Phong (1977)
    -   Cook-Torrance (1981)
    -   Ward (1992)
    -   Oren-Nayar (1994)
    -   Schlick (1994)
    -   Modified-Phong (Lafortune 1994)
    -   Lafortune (1997)
    -   Neumann-Neumann (1999)
    -   Albedo pump-up (Neumann-Neumann 1999)
    -   Ashikhmin-Shirley (2000)
    -   Kelemen (2001)
    -   Halfway Vector Disk (Edwards 2006)
    -   GGX (Walter 2007)
    -   Distribution-based BRDF (Ashikmin 2007)
    -   Kurt (2010)

  - 这些BRDF的模型可以分为如下几类：

    - 经验模型（Empirical Models）：使用基于实验提出的公式对BRDF做快速估计。
    - 数据驱动的模型（Data-driven Models）：
      - 采集真实材质表面在不同光照角度和观察角将BRDF按照实测数据建立查找表，记录在数据库中，以便于快速的查找和计算
    - 基于物理的模型（Physical-based Models）：
      - 根据物体表面材料的几何以及光学属性建立反射方程，从而计算BRDF，实现极具真实感的渲染效果

  - 1 BRDF经验模型
    - 关于BRDF的经验模型，有如下几个要点：
      -   经验模型提供简洁的公式以便于反射光线的快速计算。
      -   经验模型不考虑材质特性，仅仅提供一个反射光的粗糙近似。
      -   经验模型不一定满足物理定律，比如Helmholtz可逆性或能量守恒定律。
      -   经验模型因为其简洁和高效性被广泛运用。

    - 常见的BRDF经验模型有：
      -   Lambert漫反射模型
        - Lambert光照模型用于纯粹的漫反射表面的物体，比如磨砂的玻璃表面，观察者的所看到的反射光和观察的角度无关，这样的表面称为Lambertian
      -   Phong模型
        - 光照结果由Ambient环境光，Diffuse漫反射，Specular高光组成
      -   Blinn-Phong模型
        - Phong光照模型的改进，在表现上基本与Phong模型一致，但是性能上却优化了很多
      -   快速Phong模型
      -   可逆Phong模型

    - Lambert模型，Phong模型、Blinn-Phong模型和其改进模型都是常见的光照模型


  - 2 数据驱动的BRDF模型
    - 数据驱动的BRDF模型可以理解为，度量一个大的BRDF材质集合，并将其记录为高维向量，利用降维的方法从这些数据中计算出一个低维模型，这样基于查表的方式，可以直接找到渲染结果，省去大量的实时计算。代表工作如：A
    - Data-Driven Reflectance Model,ACMSIGGRAPH,2003 
    - [http://people.csail.mit.edu/wojciech/DDRM/index.html](http://link.zhihu.com/?target=http%3A//people.csail.mit.edu/wojciech/DDRM/index.html)

    - 另外，MERL等实验室使用各类仪器测量了上多种真实材质表面在不同光照角度和观察角度下的反射数据，并记录在数据库中
      - 如MERLBRDF Database。
      - 图8 一个名为“MERL100”的BRDF数据库
        - 其中，含50种“光滑材质”（例如金属和塑料），和50种“粗糙材质”（例如纺织物）

      - 需要注意的是，由于这些数据由于采集自真实材质，即便渲染出来的结果很真实，但缺点是没有可供调整效果的参数，无法基于这些数据修改成想要的效果，另外部分极端角度由于仪器限制，无法获取到数据，而且采样点密集，数据量非常庞大，所以并不适合游戏等实时领域，一般可用在电影等离线渲染领域，也可以用来做图形学研究，衡量其他模型的真实程度。

      - 这边提供一些数据库的链接供参考：

      -   MERLBRDF
      Database： [http://www.merl.com/brdf/](http://link.zhihu.com/?target=http%3A//www.merl.com/brdf/)

      -   MITCSAIL: [http://people.csail.mit.edu/addy/research/brdf/](http://link.zhihu.com/?target=http%3A//people.csail.mit.edu/addy/research/brdf/)

      -   CAVE
      Database:[http://www1.cs.columbia.edu/CAVE/databases/tvbrdf/about.php](http://link.zhihu.com/?target=http%3A//www1.cs.columbia.edu/CAVE/databases/tvbrdf/about.php)

# 次表面散射 Subsurface Scattering  是什么？？
  - 在真实世界中许多物体都是半透明的，比如皮肤、玉、蜡、大理石、牛奶等
    - 当光入射到透明或半透明材料表面时，一部分被反射，一部分被吸收，还有一部分经历透射。
    - 这些半透明的材质受到数个光源的透射，物体本身就会受到材质的厚度影响而显示出不同的透光性，
      - 光线在这些透射部分也可以互相混合、干涉

  - 次表面散射，Subsurface Scattering，简称SSS(又简称3S)，就是光射入半透明材质后在内部发生散射
    - 最后射出物体并进入视野中产生的现象，是指光从表面进入物体经过内部散射
      - 然后又通过物体表面的其他顶点出射的光线传递过程

# 微平面理论 Microfacet Theory  是什么？？
  - 微表面理论假设表面是由不同方向的微小细节表面，也就是微平面（microfacets）组成
    - 每一个微小的平面都会根据它的法线方向在一个方向上反射光线
    - 每个微平面的尺度都很小，眼睛看不到，但远比光波波长要长
    - 每个微平面都可以看成是完美的镜面，可以将入射光线反射到一个单独的出射的方向
      - 虽然使用法线贴图也可以描述表面的细节，但不管如何提升贴图精度仍然会有一定的缺失，更无法反馈眼睛无法看到的微小细节
      - 相对法线贴图微表面对反射的影响更多，因为粗糙的微表面会把反射光分散(D项)或者遮挡(G项)

  - 表面法线朝向光源方向和视线方向中间的微表面会反射可见光。
    - 然而，不是所有的表面法线和半角法线（half normal）相等的微表面都会反射光线，因为其中有些会被遮挡

  - 图14 微平面理论图示
    - 我们用法线分布函数（Normal DistributionFunction，简写为NDF），D(h)来描述组成表面一点的所有微表面的法线分布概率
    - 则可以这样理解：向NDF输入一个朝向h，NDF会返回朝向是h的微表面数占微表面总数的比例
      - 比如有8%的微表面朝向是h，那么就有8%的微表面可能将光线反射到v方向。

      图15
    - 由微平面组成的表面。仅红色微平面的表面法线和半矢量h对齐，能参与从入射光线向量l到视线向量v的光线反射

    - 在微观层面上不规则的表面会造成光的漫反射。例如，模糊的反射是由于光线的散射造成的。
      - 而反射的光线并不均匀，因此我们得到的高光反射是模糊的。如下图。

    - UE在编辑器环境采用GGX，但UE Mobile上采用了归一化BlinnPhong
    - 在UE4中使用法线贴图时与UDK时代有所不同，在UDK中美术为了表现物体的质感，经常会在制作完法线贴图后，人为叠加一层杂色，通过这个来控制表面反射高光的范围和效果
      - 但在PBR中由于物理层面已经模拟了物体的金属度和粗糙度。不需要添加杂色，转而使用粗糙度和金属度贴图来控制，对于高光项的BRDF方程是
---
# 菲涅尔反射 Fresnel Reflectance F  是什么？？
  - 菲涅耳方程（Fresnelequations）是一组用于描述光在两种不同折射率的介质中传播时的反射和折射的光学方程
    - 即当光入射到折射率不同的两个材质的分界面时，一部分光会被反射
    - 而我们所看到的光线会，根据我们的观察角度，以不同强度反射的现象

  - 菲涅尔反射能够真实地模拟真实世界中的反射
    - 在真实世界中，除了金属之外，其它物质均有不同程度的菲涅尔反射效果

  - 关于菲涅尔反射，一个很好的例子是一池清水
    - 从水池上笔直看下去（也就是与法线成零度角的方向）的话，我们能够一直看到池底
    - 而如果从接近平行于水面的方向看去的话，水池表面的高光反射会变得非常强以至于你看不到池底

  - 对于粗糙表面来说，在接近平行方向的高光反射也会增强但不够达到100%的强度.
    - 为何如此是因为影响菲涅尔效应的关键参数在于每个微平面的法向量和入射光线的角度
    - 而不是宏观平面的法向量和入射光线的角度
    - 因此我们在宏观层面看到的实际上是微平面的菲涅尔效应的一个平均结果。

  - a = roughness^2 , F_0 表示高光颜色

  - 根据菲涅尔反射，若你看向一个圆球，那么圆球中心的反射会较弱
    - 而靠近边缘是反射会较强。另外需注意，这种关系也受折射率影响

    - 当从很小的掠射角方向来观察这些绝缘体的时候, 反射会比较强
      - 相比之下, 金属材质的反射强弱与角度关系不大
    - 在最小的掠射角（Grazing angle）处，所有的材质都会变成完全反射,
    - 在特定的角度上，任何表面光滑的物质都可以成为完美镜面
    - 越接近边缘区域，如橙色圈内，也就是入射角为90度时，反射率无限趋近于100%。而在入射角接近0度时（蓝色圈）的反射率接近物体本身的值

    - 对于绝缘体光会在内部进行吸收和散射活动，一些折射光会通过散射，重新从入射平面反方向射出去
      - 例如, 塑料有一个白色基层, 在这个基层中通常有一些色素颗粒
        - 镜面反射出来的光线通常保持白色,原因是它并没有穿过这个材质
        - 而来自这个材质的漫反射光则穿过了这个材质并与其中的色素相互作用
    - 金属可以导电,所以一些电磁波可以激活其中的电子,而激活的电子又会使这些电磁波产生反射
      - 这种特点就导致镜面高光用到了表面颜色,同时也减少了漫反射分量
      - 因为折射进表面的光线的能量会立即被金属中的自由电子吸收，转换成电子的能量
      - 因此金属会吸收掉所有的折射光，很少会有光线在金属表面内部进行反射
      - 特别是光滑的金属基本上没有漫反射颜色，金属通常的反射率高达60%~90%
      - 而绝缘体则只有0%~20%，反射率高防止了入射光被吸收或折射，这样金属就有了闪亮的外观
  ---
# 微平面分布函数  是什么？？
  - D（h）是 微表面上的法线分布函数（Normal Distribution Function）NDF
    - 描述了单位宏表面上，微表面法线为 h 的概率密度，即微表面法向量 m 与向量 h 相同的比例
    - 在大多数表面上，微表面的方向不是均匀分布的
    - 微表面法线，接近，宏表面法线的频率越高就越光滑
  - D决定了高光的大小、亮度，形状和衰减速度等等
    - 出现在图形文献中的法线分布函数都是类似高斯分布形式
    - 具有某种“粗糙度”或方差参数
    - 随着表面粗糙度的降低，在整个宏表面法线 n 周围的微表面法线 m 密度增加
      - 并且 D(h) 的值可能变得非常高
# 几何衰减因子 是什么？？
  - Geometry项G，来代表反射光的可见度
    - 遮挡项在某种程度上抵消了microfacet方程中的分母 (n * l)(n* v) ，经常会和其他参数合并成为V项



# 基于物理的BRDF     怎么表示？？
  - 基于物理的BRDF这个出发点，看看从物理的角度，diffuse和specular来自哪里
    - 什么决定了材质的diffuse和specular颜色
    - diffuse是光线折射入物体后，在内部经过散射，重新穿出表面的光
    - specular是光线在物体表面直接反射的结果，并没有经过转换，还是原来的光
    - 对于金属（导体）来说，折射入物体的光子会被完全吸收掉
      - diffuse永远是0，材质的颜色来自于 specular
    - 对于非金属来说，材质的颜色一部分来自于diffuse，一部分来自于specular
  - 颜色分布
    - 控制diffuse和specular的颜色，就一定能在渲染上复现出金属（导体）和塑料（绝缘体）的区别， 甚至介于其间的半导体

    - 找一些现实中的材料，看看它们的diffuse和specular颜色分布究竟如何
      - 前面说到金属的diffuse = 0，所以你看到的金属颜色，就是它的specular
      - 同样是specular颜色，非金属则显得弱了很多
        - 塑料、玻璃、水这些常见的绝缘体，specular都低于0.04
      - 绝缘体的specular都是单色的
        - 各个通道的光都会被等同地反射，而不像金属那样有着不同的吸收偏好
      - 半导体，介于两者之间
# GBuffer 在BRDF  中如何应用？？
  - 按照理论，需要放diffuse和specular颜色，则需要6个通道
    - 完全超出了GBuffer可以承受的范围
  - GBuffer里留给diffuse和specular颜色的通道只有4个
    - 所以游戏引擎常见的做法是，保存个diffuse颜色和specular的亮度
      - 就当作specular是单色的来处理
      - 这样显然对于金属不利，也是为什么会画面看着都像塑料的重要原因

# Lambertian漫反射模型 是什么？？
  - Lambertian反射也称做完全漫反射
    - 表示各种方向上辐射亮度都一样的反射面
      - 这是一种理想情况，现实中是不存在完全漫反射的
# Cook-Torrance BRDF模型  是什么？？
    - Cook-Torrance模型将物理学中的菲涅尔反射引入了图形学，实现了比较逼真的效果
    - F为菲涅尔反射函数( Fresnel 函数 )
    - G为阴影遮罩函数（Geometry Factor，几何因子），即未被shadow或mask的比例
    - D为法线分布函数(NDF)
  - Ward BRDF模型
    - 各项同性（Isotropic）的BRDF

反射不受与给定表面法向夹角的约束

随机表面微结构

各向异性（Anisotropic）的BRDF

反射比随着与某个给定的表面法向之间的夹角而变化

图案的表面微结构

金属丝，绸缎，毛发等

Phong和Cook-Torrance BRDF模型都不能处理各项异性的效果，Ward模型却可以。

Ward模型Ward模型介绍了更一般的表面法向表达方式：通过椭圆体（ellipsoids）这种允许各向异性反射的形式来表达。

然而，由于没有考虑菲涅耳因子（Fresnel factor）和几何衰减因子（geometric attenuation factor），该模型更像是一种经验模型，但还是属于基于物理的BRDF模型
# BRDF引申
  - BSSRDF
    - BRDF只是更一般方程的一种近似,这个方程就是BSSRDF（Bidirectional scattering-surface reflectance distribution function，双向表面散反射分布函数）。BSSRDF描述了射出辐射率与入射通量之间的关系，BSSRDF函数通过把入射和出射位置作为函数的输入，从而来包含这些现象，它描述了沿入射方向从物体表面的一点到另外一点，最后顺着出射方向出去的光线的相对量。注意，这个函数还考虑了物体表面的一点到另外一点，最好顺着出射方向出去的光线相对量。注意，这个函数还考虑了物体表面不一致的情况，因为随着位置的变化，反射系数也会发生变化。在实时绘制中，物体表面上的位置可以用来获取颜色纹理、光泽度，以及凹凸纹理图等信息
  - SBRDF(SVBRDF)
    - 一个捕获基于空间位置BRDF变化的函数被称为空间变化的BRDF（Spatially Varying BRDF ,SVBRDF）或称空间BRDF，空间双向反射分布函数（Spatial BRDF ，SBRDF）
  - BTDF与BSDF
    - 即使一般的BSSRDF函数，无论其多么复杂，仍然忽略了现实世界中非常重要的一些变量，比如说光的偏振。此外，也没有处理穿过物体表面的光线传播，只是对反射情况进行了处理。为了处理光线传播的问题，对物体表面定义了两个BRDF和两个BTDF（T表示传播“Transmittance”），每侧各有一个，这样就组成了BSDF（S表示散射“Scattering”）



---
##  基于图像的渲染技术总结
# 光谱渲染 The Rendering Spectrum 是什么？？
  - 众所周知，渲染的目的就是在屏幕上渲染出物体，至于如何达到结果，主要依赖于用户的选择
    - 而用多边形将三维物体显示在屏幕上，并非是进行三维渲染的唯一方法，也并非是最合适的方法。
  - 多边形具有从任何视角以合理的方式表示对象的优点，当移动相机的时候，物体的表示可以保持不变
    - 但是，当观察者靠近物体的时候，为了提高显示质量，往往希望用比较高的细节层次来表示模型
    - 与之相反，当物体位于比较远的地方时，就可以用简化形式来表示模型。
    - 这就是细节层次技术(Level Of Detail,LOD)。
      - 使用LOD技术主要目的是为了加快场景的渲染速度

    - 还有很多技术可以用来表示物体逐渐远离观察者的情形
      - 比如，可以用图像而不是多边形来表示物体，从而减少开销，加快渲染速度
      - 另外，单张图片可以很快地被渲染到屏幕上，用来表示物体往往开销很小
# 固定视角的渲染 Fixed-View Rendering 是什么？？
  - 通过将复杂几何模型转换为可以在多帧中重复使用的一组简单的buffer来节省大量渲染时间与性能。
    - 对于复杂的几何和着色模型，每帧去重新渲染整个场景很可能是昂贵的
      - 可以通过限制观看者的移动能力来对渲染进行加速。
      - 最严格的情况是相机固定在位置和方位，即根本不移动。
      - 而在这种情况下，很多渲染可以只需做一次。

      - 例如，想象一个有栅栏的牧场作为静态场景，一匹马穿过它。
        - 牧场和栅栏渲染仅一次，存储其颜色和Z缓冲区。
        - 每帧将这些buffer复制到可显示的颜色和Z缓冲中。
        - 为了获得最终的渲染效果，马本身是需要渲染的。
        - 如果马在栅栏后面，存储和复制的z深度值将把马遮挡住。
          - 请注意，在这种情况下，马不能投下阴影，因为场景无法改变
          - 可以进行进一步的处理，例如，可以确定出马影子的区域，根据需求进行处理。
          - 关键是对于要显示的图像的颜色何时或如何设置这点上，是没有限制的。
          - 固定视角的特效（Fixed-View Effects），可以通过将复杂几何模型转换为可以在多帧中重复使用的一组简单的buffer来节省大量时间

在计算机辅助设计（CAD）应用程序中，所有建模对象都是静态的，并且在用户执行各种操作时，视图不会改变。一旦用户移动到所需的视图，就可以存储颜色和Z缓冲区，以便立即重新使用，然后每帧绘制用户界面和突出显示的元素。 这允许用户快速地注释，测量或以其他方式与复杂的静态模型交互。通过在G缓冲区中存储附加信息，类似于延迟着色的思路，可以稍后执行其他操作。 例如，三维绘画程序也可以通过存储给定视图的对象ID，法线和纹理坐标来实现，并将用户的交互转换为纹理本身的变化。

一个和静态场景相关的概念是黄金线程(Golden Thread)或自适应（Adaptive Refinement）渲染。其基本思想是，当视点与场景不运动时，随着时间的推移，计算机可以生成越来越好的图像，而场景中的物体看起来会更加真实，这种高质量的绘制结果可以进行快速交换或混合到一系列画面中。这种技术对于CAD或其他可视化应用来说非常有用。而除此之外，还可以很多不同的精化方法。一种可能的方法是使用累积缓冲器（accumulation buffer）做抗锯齿（anti- aliasing），同时显示各种累积图像。另外一种可能的方法是放慢每像素离屏着色（如光线追踪，环境光遮蔽，辐射度）的速度，然后渐进改进之后的图像。

在RTR3书的7.1节介绍一个重要的原则，就是对于给定的视点和方向，给定入射光，无论这个光亮度如何计算或和隔生成这个光亮度的距离无关。眼睛没有检测距离，仅仅颜色。在现实世界中捕捉特定方向的光亮度可以通过简单地拍一张照片来完成。

QuickTime VR是由苹果公司在1995年发布的VR领域的先驱产品，基本思路是用静态图片拼接成360度全景图。QuickTime VR中的效果图像通常是由一组缝合在一起的照片，或者直接由全景图产生。随着相机方向的改变，图像的适当部分被检索、扭曲和显示。虽然这种方法仅限于单一位置，但与固定视图相比，这种技术具有身临其境的效果，因为观看者的头部可以随意转动和倾斜。

Kim，Hahn和Nielsen提出了一种有效利用GPU的柱面全景图，而且通常，这种全景图也可以存储每个纹素的距离内容或其他值，以实现动态对象与环境的交互。

如下的三幅图，便是是基于他们思想的全景图（panorama），使用QuickTime VR来渲染出的全景视野范围。其中，第一幅是全景图原图，后两幅图是从中生成的某方向的视图。注意观察为什么这些基于柱面全景图的视图，没有发生扭曲的现象
# 天空盒 Skyboxes  是什么 ？？ 怎么渲染？？
  - 对于一些远离观众的物体，观众移动时几乎没有任何视差效果。
    - 换言之，如果你移动一米，甚至一千米，一座遥远的山本身看起来通常不会有明显的不同
    - 当你移动时，它可能被附近的物体挡住视线，但是把那些物体移开，山本身看起来也依旧一样
  - 环境贴图（environment map）可以代表本地空间入射光亮度。
    - 虽然环境贴图通常用于模拟反射，但它们也可以直接用来表示环绕环境的远处物体。
    - 任何独立于视图的环境地图表示都可以用于此目的；
    - 立方体贴图（cubic maps）是最为常见的一种环境贴图。
    - 环境贴图放置在围绕着观察者的网格上，并且足够大以包含场景中所有的对象。
    - 且网格的形状并不重要，但通常是立方体贴图。如下图，在该图所示的室内环境更像是一个QuickTime VR全景的无约束版本。
    - 观众可以在任何方向观察这个天空盒，得到很好的真实体验。
    - 但同样，任何移动都会破坏这个场景产生的真实感错觉，因为移动的时候，并不存在视差
  - 环境贴图通常可以包含相对靠近反射对象的对象。因为我们通常并没有多精确地去在乎反射的效果，所以这样的效果依然非常真实。而由于视差在直接观看时更加明显，因此天空盒通常只包含诸如太阳，天空，远处静止不动的云和山脉之类的元素
  - 图9 玻璃球折射和反射效果的一个立方体环境贴图，这个map本身用可作天空盒。

  - 为了使天空盒看起来效果不错，立方体贴图纹理分辨率必须足够，即每个屏幕像素的纹理像素。
    - 必要分辨率的近似值公式：
    - 其中，fov表示视域。该公式可以从观察到立方体贴图的表面纹理必须覆盖90度的视域（水平和垂直）的角度推导出
      - 并且应该尽可能隐藏好立方体的接缝处，最好是能做到无缝的衔接，使接缝不可见。
      - 一种解决接缝问题的方法是，使用六个稍微大一点的正方形，来形成一个立方体
        - 这些正方形的每个边缘处彼此相互重叠，相互探出。
        - 这样，可以将邻近表面的样本复制到每个正方形的表面纹理中，并进行合理插值
# 光场渲染 Light Field Rendering  是什么 ？？怎么 渲染？？
  - 所谓光场（Light Field），可以理解为空间中任意点发出的任意方向的光的集合
    - 光场渲染（Light Field Rendering），可以理解为在不需要图像的深度信息或相关性的条件下
      - 通过相机阵列或由一个相机按设计的路径移动，把场景拍摄下来作为输出图像集
    - 对于任意给定的新视点，找出该视点邻近的几个采样点进行简单的重新采样和插值，就能得到该视点处的视图

# 精灵与层 Sprites and Layers  是什么 ？？怎么 渲染？？
  - 最基本的基于图像的渲染的图元之一便是精灵（sprite）
    - 精灵（sprite）是在屏幕上移动的图像，例如鼠标光标
    - 精灵不必具有矩形形状，而且一些像素可以以透明形式呈现
    - 对于简单的精灵，屏幕上会显示一个一对一的像素映射
    - 存储在精灵中的每个像素将被放在屏幕上的像素中
      - 可以通过显示一系列不同的精灵来生成动画
  - 图13 基于Sprite层级制作的《雷曼大冒险》@UBISOFT

  - 更一般的精灵类型是将其渲染为应用于总是面向观看者的多边形的图像纹理
    - 图像的Alpha通道可以为sprite的各种像素提供全部或部分透明度
    - 这种类型的精灵可以有一个深度，所以在场景本身，可以顺利地改变大小和形状
    - 一组精灵也可以用来表示来自不同视图的对象
    - 对于大型物体，这种用精灵来替换的表现效果会相当弱，因为从一个精灵切换到另一个时，会很容易穿帮。
    - 也就是说，如果对象的方向和视图没有显着变化，则给定视图中的对象的图像表示可以对多个帧有效
    - 而如果对象在屏幕上足够小，存储大量视图，即使是动画对象也是可行的策略

    - 考虑场景的一种方法是将其看作一系列的层（layers），而这种思想也通常用于二维单元动画。
      - 每个精灵层具有与之相关联的深度
      - 通过这种从前到后的渲染顺序，我们可以渲染出整个场景而无需Z缓冲区，从而节省时间和资源
# 公告板 Billboarding  是什么 ？？怎么 渲染？？
  - 我们将根据观察方向来确定多边形面朝方向的技术叫做公告板（Billboarding，也常译作布告板）
    - 而随着观察角度的变化，公告板多边形的方向也会根据需求随之改变
    - 与alpha纹理和动画技术相结合，可以用公告板技术表示很多许多不具有平滑实体表面的现象
      - 比如烟，火，雾，爆炸效果，能量盾（Energy Shields），水蒸气痕迹，以及云朵等。
      - 如下文中贴图的，基于公告板渲染出的云朵
  - 图16 给定表面的法线向量n和近似向上方向的向量u，通过创建一组由三个相互垂直的向量，就可以确定公告板的方向
    - 其中，左图是互相垂直的u和n。中图是r向量通过u和n的叉乘得到，因此同时垂直于u和n
      - 而在右图中，对固定向量n和r进行叉乘就可以得到与他们都垂直的的向上向量u’

  - 有三种不同类型的Billboard，分别是：
    - Screen-Aligned Billboard 对齐于屏幕的公告板
    - World-Oriented Billboard 面向世界的公告板
    - Axial Billboard 轴向公告板
    - 其中：
      - Screen-Aligned Billboard的n是镜头视平面法线的逆方向,u是镜头的up。
      - Axial Billboard的u是受限制的Axial, r = u* n,(n是镜头视平面法线的逆方向,或,视线方向的逆方向),最后再计算一次n' = r * u,即n'才是最后可行的代入M的n,表达了受限的概念。
      - World-orientedbillboard就不能直接使用镜头的up做up,因为镜头roll了,并且所画的billboard原本是应该相对世界站立的,按Screen-Aligned的做法就会随镜头旋转,所以此时应该r = u * n(u是其在世界上的up,n是镜头视线方向的逆方向),最后再计算一次u = r * n,即u'才是最后的up,即非物体本身相对世界的up,亦非镜头的up。

    - 所以公告板技术是一种看似简单其实较为复杂的技术,它的实现变种较多。归其根本在于：
      - View Oriented / View plane oriented的不同
      - Sphere/ Axial的不同
      - Cameraup / World up的不同
      - 如View Oriented 和View plane oriented的不同，得到的公告板效果就完全不同
# 粒子系统 Particle System  是什么 ？？怎么 渲染？？
  - 粒子系统（Particle System）是一组分散的微小物体集合，其中这些微小物体按照某种算法运动。
  - 粒子系统的实际运用包括模拟火焰，烟，爆炸，流水，树木，瀑布，泡沫，旋转星系和其他的一些自然现象。
  - 粒子系统并不是一种渲染形式，而是一种动画方法
    - 这种方法的思想是值粒子的生命周期内控制他们的产生，运动，变化和消失。

  - 可以用一条线段表示一个实例，另外，也可以使用轴向公告板配合粒子系统，显示较粗的线条

  - 除了爆炸，瀑布，泡沫以及其他现象以外，还可以使用粒子系统进行渲染。
    - 例如，可以使用粒子系统来创建树木模型，也就是表示树木的几何形状，当视点距离模型较近时，就会产生更多的粒子来生成逼真的视觉效果

# 替代物 Impostors  是什么 ？？怎么 渲染？？
  - 作为一种公告板技术，替代物（Impostors）是通过从当前视点将一个复杂物绘制到一幅图像纹理上来创建的
    - 其中的图像纹理用于映射到公告板上，渲染过程与替代物在屏幕上覆盖的像素点数成正比
      - 而不是与顶点数或者物体的复杂程度成正比
    - 替代物可以用于物体的一些实例上或者渲染过程的多帧上，从而使整体性能获得提升
  - 另外，Impostors和Billboard的纹理还可以结合深度信息（如使用深度纹理和高度纹理）进行增强
    - 如果对Impostors和Billboard增加一个深度分量，就会得到一个称为深度精灵（depth sprite）或者nailboard（译作钉板，感觉很奇怪）的相关绘制图元
    - 也可以对Impostors和Billboard的纹理做浮雕纹理映射（relief texture mapping）

    - 关于Impostors，一篇很好的文章是William Damon的《Impostors Made Easy》，有进一步了解兴趣的朋友可以进行延伸阅读：
    - https://software.intel.com/en-us/articles/impostors-made-easy
# 公告板云 Billboard Clouds   是什么 ？？怎么 渲染？？
  - 使用Imposters的一个问题是渲染的图像必须持续地面向观察者。
  - 如果远处的物体正在改变方向，则必须重新计算Imposters的朝向。
  - 而为了模拟更像他们所代表的三角形网格的远处物体，D´ecoret等人提出了公告板云（Billboard Clouds）的想法
    - 即一个复杂的模型通常可以通过一系列的公告板集合相互交叉重叠进行表示。
    - 我们知道，一个真实物体可以用一个纸模型进行模拟，而公告板云可以比纸模型更令人信服
      - 比如公告板云可以添加一些额外的信息，如法线贴图、位移贴图和不同的表面材质
      - 另外，裂纹沿裂纹面上的投影也可以由公告板进行处理。
      - 而D´ecoret等人也提出了一种在给定误差容限内对给定模型进行自动查找和拟合平面的方法。

# 图像处理 Image Processing  是什么 ？？怎么 渲染？？
  - 图像处理的过程，一般在像素着色器中进行，因为在像素着色器中，可以很好地将渲染过程和纹理结合起来
    - 而且在GPU上跑像素着色器，速度和性能都可以满足一般所需

    - 一般而言，首先需要将场景渲染成2D纹理或者其他图像的形式，再进行图像处理
      - 这里的图像处理，往往指的是后处理（post effects）
      - 而下文将介绍到的颜色校正（Color Correction）、色调映射（Tone Mapping）
        - 镜头眩光和泛光（Lens Flare and Bloom）、景深（Depth of Field）、运动模糊（Motion Blur），一般而言都是后处理效果
# 颜色校正 Color Correction   是什么 ？？怎么 渲染？？
  - 色彩校正(Color correction)是使用一些规则来转化,给定的现有图像的,每像素颜色到其他颜色的一个过程
    - 颜色校正有很多目的，例如模仿特定类型的电影胶片的色调
    - 在元素之间提供一致的外观，或描绘一种特定的情绪或风格
    - 一般而言，通过颜色校正，游戏画面会获得更好的表现效果
  - 图27 左图是准备进行颜色校正的原图。右图是通过降低亮度，使用卷积纹理（Volume Texture），得到的夜间效果。\@Valve

  - 颜色校正通常包括将单个像素的RGB值作为输入，并向其应用算法来生成一个新的RGB
    - 颜色校正的另一个用途是加速视频解码，如YUV色彩空间到RGB色彩空间的转换
    - 基于屏幕位置或相邻像素的更复杂的功能也可行，但是大多数操作都是使用每像素的颜色作为唯一的输入。

    - 对于一个计算量很少的简单转换，如亮度的调整，可以直接在像素着色器程序中基于一些公式进行计算，应用于填充屏幕的矩形。
    - 而评估复杂函数的通常方法是使用查找表（Look-Up Table，LUT）
      - 由于从内存中提取数值经常要比复杂的计算速度快很多，所以使用查找表进行颜色校正操作，速度提升是很显著的。
# 色调映射 Tone Mapping   是什么 ？？怎么 渲染？？
  - 计算机屏幕具有特定的亮度范围，而真实图像具有更巨大的亮度范围
    - 色调映射（Tonemapping），也称为色调复制（tone reproduction），便是将宽范围的照明级别拟合到屏幕有限色域内的过程
      - 色调映射与表示高动态范围的HDR和HDRI密切相关：
 - 本质上来讲，色调映射要解决的问题是进行大幅度的对比度衰减将场景亮度变换到可以显示的范围
        - 同时要保持图像细节与颜色等表现原始场景的重要信息。
      - 根据应用的不同，色调映射的目标可以有不同的表述
        - 在有些场合，生成“好看”的图像是主要目的，而在其它一些场合可能会强调生成尽可能多的细节或者最大的图像对比度
        - 在实际的渲染应用中可能是要在真实场景与显示图像中达到匹配，尽管显示设备可能并不能够显示整个的亮度范围。

# HDR   是什么 ？？怎么 渲染？？
  - 是High-Dynamic Range（高动态范围）的缩写，可以理解为一个CG的概念，常出现在计算机图形学与电影、摄影领域中
  - HDRI是High-Dynamic Range Image的缩写，即HDR图像，高动态范围图像
  - 而实际过程中，HDR和HDRI两者经常会被混用，都当做高动态范围成像的概念使用，这也是被大众广泛接受的。

# 镜头眩光和泛光 Lens Flare and Bloom   是什么 ？？怎么 渲染？？
  - 镜头眩光（Lens flare）是由于眼睛的晶状体或者相机的透镜直接面对强光所产生的一种现象
    - 由一圈光晕（halo）和纤毛状的光环（ciliary corona）组成
  - 光晕的出现是因为透镜物质（如三棱镜）对不同波长光线折射数量的不过而造成的
    - 看上去很像是光周围的一个圆环，外圈是红色，内圈是紫红色
    - 纤毛状的光环源于透镜的密度波动，看起来像是从一个点发射出来的光线
  - Lens flare是近来较为流行的一种图像效果，自从我们认识到它是一种实现真实感效果的技术后，计算机便开始模拟此效果
    - 图31 镜头眩光效果 @WatchDogs

  - 泛光（Bloom）效果，是由于眼睛晶状体和其他部分的散光而产生，在光源附近出现的一种辉光
    - 在现实世界中，透镜无法完美聚焦是泛光效果的物理成因；
    - 理想透镜也会在成像时由于衍射而产生一种名为艾里斑的光斑。

    - 常见的一个误解便是将HDR和Bloom效果混为一谈
      - Bloom可以模拟出HDR的效果，但是原理上和HDR相差甚远
      - HDR实际上是通过映射技术，来达到整体调整全局亮度属性的
        - 这种调整是颜色，强度等都可以进行调整，而Bloom仅仅是能够将光照范围调高达到过饱和，也就是让亮的地方更亮
          - 不过Bloom效果实现起来简单，性能消耗也小，却也可以达到不错的效果
# 景深 Depth of Field    是什么 ？？怎么 渲染？？
  - 也叫焦点范围（focus range）或有效焦距范围（effective focus）
    - 是指场景中最近和最远的物体之间出现的可接受的清晰图像的距离
    - 换言之，景深是指，相机对焦点，前后相对清晰的，成像范围
    - 在相机聚焦完成后，在焦点前后的范围内都能形成清晰的像，这一前一后的距离范围，便叫做景深。
  - 图34 摄影中典型的景深效果

  - 虽然透镜只能够将光聚到某一固定的距离，远离此点则会逐渐模糊
    - 但是在某一段特定的距离内，影像模糊的程度是肉眼无法察觉的，这段距离称之为景深
    - 当焦点设在超焦距处时，景深会从超焦距的一半延伸到无限远，对一个固定的光圈值来说，这是最大的景深。

    - 景深通常由物距、镜头焦距，以及镜头的光圈值所决定（相对于焦距的光圈大小）
      - 除了在近距离时，一般来说景深是由物体的放大率以及透镜的光圈值决定
      - 固定光圈值时，增加放大率，不论是更靠近拍摄物或是使用长焦距的镜头，都会减少景深的距离；
      - 减少放大率时，则会增加景深。如果固定放大率时，增加光圈值（缩小光圈）则会增加景深；减小光圈值（增大光圈）则会减少景深。

      - 景深的效果在计算机图形学中应用广泛，电影，游戏里面经常会利用景深特效来强调画面重点。相应的，已经有了很多成熟的算法在不同的渲染方法，而光栅化可以很高效的实现现有的景深算法。
# 动态模糊 Motion Blur  是什么 ？？怎么 渲染？？
  - 现实世界中，动态模糊（Motion Blur，或译为动态模糊)
    - 是因为相机或者摄影机的快门时间内物体的相对运动产生的
    - 在快门打开到关上的过程中，感光材料因为受到的是物体反射光持续的照射成像
    - 即在曝光的这个微小时间段内，对象依然在画面中移动，感光材料便会记录下这段时间内物体运动的轨迹，产生运动模糊。

    - 我们经常在电影中看到这种模糊，并认为它是正常的
      - 所以我们期望也可以在电子游戏中看到它，以带给游戏更多的真实感

    - 若无运动模糊，一般情况下，快速移动的物体会出现抖动，在帧之间的多个像素跳跃
      - 这可以被认为是一种锯齿，但可以理解为基于时间的锯齿，而不是基于空间的锯齿
      - 在这个意义上，运动模糊可以理解为是一种时间意义上的抗锯齿。

    - 正如更高的显示分辨率可以减少但不能消除锯齿，提高帧速率并不能消除运动模糊的需要
      - 而视频游戏的特点是摄像机和物体的快速运动，所以运动模糊可以大大改善画面的视觉效果
      - 而事实表明，带运动模糊的30 FPS画面，通常看起来比没有带运动模糊的60 FPS画面更出色。
  - 图37 Motion Blur效果 @GTA5

  - 在计算机绘制中产生运动模糊的方法有很多种
    - 一个简单但有限的方法是建模和渲染模糊本身。

  - 实现运动模糊的方法大致分3种：
    - 1、直接渲染模糊本身
      - 通过在对象移动之前和之后添加几何体来完成，并通过次序无关的透明，避免Alpha混合。
    - 2、基于累积缓冲区（accumulationbuffer）
      - 通过平均一系列图像来创建模糊
    - 3、基于速度缓冲器（velocity buffer）
      - 目前这个方法最为主流
      - 创建此缓冲区，需插入模型三角形中每个顶点的屏幕空间速度
      - 通过将两个建模矩阵应用于模型来计算速度，一个用于最后一个帧，一个用于当前模型
      - 顶点着色器程序计算位置的差异，并将该向量转换为相对的屏幕空间坐标
      - 图10.34显示了速度缓冲器及其结果。
  - 图38 Motion Blur效果 @Battlefield4

  - 运动模糊对于由摄像机运动而变得模糊的静态物体来说比较简单，因为往往这种情况下不需要速度缓冲区
    - 如果需要的是摄像机移动时的运动感，可以使用诸如径向模糊（radial blur）之类的固定效果。如下图。
# 体渲染 Volume Rendering   是什么 ？？怎么 渲染？？
  - 体渲染（Volume Rendering），又称立体渲染，体绘制
    - 是用于显示，离散三维采样数据集的，二维投影的技术
    - 体渲染技术中的渲染数据一般用体素（Volumeric Pixel，或Voxel）来表示，每个体素表示一个规则空间体
    - 例如，要生成人头部的医学诊断图像（如CT或MRI）
      - 同时生成256 x256个体素的数据集合
      - 每个位置拥有一个或者多个值
        - 则可以将其看做三维图像
    - 因此，体渲染也是基于图像的渲染技术中的一种
    - 体渲染技术流派众多，常见的流派有：
      - 体光线投射Volume ray casting
      - 油彩飞溅技术Splatting
      - 剪切变形技术Shear warp
      - 基于纹理的体绘制Texture-based volume rendering
---

---
##  全局光照:光线追踪、路径追踪与GI
  -


# 非真实感渲染(NPR)
# 渲染加速算法
# 渲染管线优化方法
