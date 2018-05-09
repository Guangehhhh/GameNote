## FFT 算法

---
- 雅克比行列式
  - Tessendorf 利用雅各比行列式来检测海面网格点发生重叠的程度，
    - 若重叠较大产生失真，则将λ 设置为一个较小的值来避免这个问题
    - 这种重叠实际上可能是一种标志喷雾、泡沫、碎浪的产生的有效方法
    - 为了使用这个重叠的区域，我们需要一种简单和快速的方法来决定这种效果是否正在发生
    - 有一种简单的雅克比行列式形式的测试检测从 x 到 x + λD(x, t)的变换
    - 雅克比行列式是一种对变换唯一性的测量
      - 当位移为 0 时，雅克比行列式为 1，当位移存在时，雅克比行列式如下
        - J(X)=JxxJyy- JxyJyx
  - 雅克比行列式标志了重叠波的存在，因为它的值在重叠区域内是小于 0 的
    - 汹涌海浪表面的折叠部分对应区域为 J<0
    - 针对这个信息，我们可以相对简单地提取这地区域并利用它产生其他效果
      - 如海浪的白盖现象和第四章的区分波峰波谷的着色效果
      - 图 2-5 展示了未经水平位移高度场构建的海面和经过水平位移的峭波海面
        - 注意这两片海面使用的是同一个垂直位置高度场

# FFT海洋
  - 基于海浪谱模型生成的海面高度场数据块可以无缝拼接为一整片海洋
    - 该方法速度快，而且符和实时性要求
    - 适用于风力驱动的大规模深海海面的海浪建模，因此我们选择海浪谱模型
    - 海面上网格某一顶点的水平位置向量 X=(x，z)
      - 构成海浪谱的各种成分波的波矢
  - 1海面波浪高度场的构建
    - 确定算法所需要的参数：模拟一片海域的网格尺寸 Lx和Ly
      - x 和 z 方向上的网格采样个数 M、N
      - 风向量 ，陡峭因子λ ，以及用来判断是否生成陡峭波的布尔变量 Choppy
  - 2生成两个高斯随机数
  - 3计算 Ph'(k)，根据 Phillips 谱函数计算每一点的 Ph'(k)值
    - 海面波浪运动所产生的波幅看作统计不变的、单一的、且符合正态分布的频谱
  - 4计算 h0(k)，将第三步得到的h0(k)值代入公式
    - 傅里叶变换的高度分量，决定海面波浪形状
    - 傅里叶分量的幅值通过 Phillips 频谱和随机数得到
  - 5计算 h(K,t) ，根据 FFT 的性质有 h0(k)+h0(-k)
    - 通过快速傅里叶变换利用波数域计算出位置域
    - 进一步可以用位置和时间计算出波幅
  - 6计算 h(X,t)，利用 FFT 逆变换的快速算法求出这 X 点 t 时刻的瞬时波高
    - 海浪高度 是一个关于时间 t和水平面网格坐标位置 X 的一个随机变量
    - 它由一系列具有不同波振幅、波相位和波速的正弦和余弦波叠加而成
  - 7若 choppy 为 TRUE，则根据公式生成水面网格点的平移向量
  - 8循环以上计算方法，求出 t 时刻海平面Lx*Ly 的高度场
  - 9将所生成的高度场加入到生成的投影网格中，进行渲染

----
## GLSL  
# 使用缓冲区
  - 创建缓冲区：glGenBufffers
  - 绑定缓冲区：glBindBuffer
  - 填充缓冲区：glBufferData
  -
# 多次渲染传递.
  - 在一些通用运算中，我们会希望把前一次运算结果传递给下一个运算用来作为后继运算的输入变量
  - 但是在GPU中，一个纹理不能同时被读写，这就意味着我们要创建另外一个渲染通道，并给它绑定不同的输入输出纹理，甚至要生成一个不同的运算内核
  - 有一种非常重要的技术可以用来解决这种多次渲染传递的问题，让运算效率得到非常好的提高，这就是“乒乓”技术。

# 乒乓技术
  - 乒乓技术，是一个用来把渲染输出转换成为下一次运算的输入的技术
  - 在本文中（y_new = y_old + alpha * x） ，这就意味我们要切换两个纹理的角色
      - y_new 和 y_old
  - 有三种可能的方法来实现这种技术（看一下以下这篇论文Simon Green's FBO slides ，这是最经典的资料了）：

    - 为每个将要被用作渲染输出的纹理指定一个绑定点
      - 并使用函数glBindFramebufferEXT()来为每个渲染通道绑定一个不同的FBO.
    - 只使用一个FBO,但每次通道渲染的时候
      - 使用函数glBindFramebufferEXT()来重新绑定渲染的目标纹理
    - 使用一个FBO和多个绑定点，使用函数glDrawBuffer()来交换它们
# 理解GPU编程的基础
//生成并绑定一个FBO，也就是生成一个离屏渲染对像
GLuint fb;
glGenFramebuffersEXT(1,&fb);
glBindFramebufferEXT(GL_FRAMEBUFFER_EXT,fb);
// 生成两个纹理，一个是用来保存数据的纹理，一个是用作渲染对像的纹理
GLuint tex,fboTex;
glGenTextures (1, &tex);
glGenTextures (1, &fboTex);
glBindTexture(GL_TEXTURE_RECTANGLE_ARB,fboTex);
// 设定纹理参数
glTexParameteri(GL_TEXTURE_RECTANGLE_ARB,
GL_TEXTURE_MIN_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_RECTANGLE_ARB,
GL_TEXTURE_MAG_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_RECTANGLE_ARB,
GL_TEXTURE_WRAP_S, GL_CLAMP);
glTexParameteri(GL_TEXTURE_RECTANGLE_ARB,
GL_TEXTURE_WRAP_T, GL_CLAMP);
// 这里在显卡上分配FBO纹理的贮存空间，每个元素的初始值是0；
glTexImage2D(GL_TEXTURE_RECTANGLE_ARB,0,GL_RGBA32F_ARB,
texSize,texSize,0,GL_RGBA,GL_FLOAT,0);
// 分配数据纹理的显存空间
glBindTexture(GL_TEXTURE_RECTANGLE_ARB,tex);

glTexParameteri(GL_TEXTURE_RECTANGLE_ARB,
GL_TEXTURE_MIN_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_RECTANGLE_ARB,
GL_TEXTURE_MAG_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_RECTANGLE_ARB,
GL_TEXTURE_WRAP_S, GL_CLAMP);
glTexParameteri(GL_TEXTURE_RECTANGLE_ARB,
GL_TEXTURE_WRAP_T, GL_CLAMP);

glTexEnvf(GL_TEXTURE_ENV,GL_TEXTURE_ENV_COLOR,GL_DECAL);
glTexImage2D(GL_TEXTURE_RECTANGLE_ARB,0,GL_RGBA32F_ARB,
texSize,texSize,0,GL_RGBA,GL_FLOAT,0);

//把当前的FBO对像，与FBO纹理绑定在一起
glFramebufferTexture2DEXT(GL_FRAMEBUFFER_EXT,
GL_COLOR_ATTACHMENT0_EXT,
GL_TEXTURE_RECTANGLE_ARB,fboTex,0);
// 把本地数据传输到显卡的纹理上。
glBindTexture(GL_TEXTURE_RECTANGLE_ARB,tex);

glTexSubImage2D(GL_TEXTURE_RECTANGLE_ARB,0,0,0,texSize,texSize,
GL_RGBA,GL_FLOAT,data);
# GLSL Texture 流程
  - glActiveTexture(GL_TEXTURE0 + 1); //选择纹理单元1
  - glBindTexture(GL_TEXTURE_CUBE_MAP, texture);
  - glUniform1i(glGetUniformLocation(program, name), 1);//通知采样器，让其知道它代表的时纹理单元1
# GLSL Shader 流程
  - shader代码可以用文本文件保存，可以在程序中用字符串保存
    - 但最终还是必须在程序中以字符串的形式传入Shader对象中；
  - 1创建Shader对象glCreateShader
    - 成功时返回非0的无符号Handle值，指涉Shader对象）；
  - 2把shader代码传入shader对象glShaderSource
    - 注意此函数的参数，字符流地址参量是GLchar*，不支持宽字符
      - 执行后可删除内存上保存的shader代码字符串副本）；
  - 3 编译Shader对象——正如我们编译任何代码一样（glCompileShader
    - 应该以GL_COMPILE_STATUS为参调用glGetShaderiv检查编译是否成功
    - 如果代码出现问题会在这个阶段报错
      - debug时可用glGetError查看更具体的错误类型）；
      - 按上步骤把一个“过程”中需要的各类型shader编译好（通常每种类型最多建一个shader对象——因为对于一个渲染管道（流程）来说，顶点、单几何体、像素都只处理一遍，不可能返回。
      - shaders只是插入这个流程中取代对应的固定管道处理的“外挂”[在抹除固定管道处理、全shader时代来临之前可以这么说]，在一个渲染流程中起作用的shaders姑且统称为一个shader过程）；
  - 4创建这么一个shaderProgram对象glCreateProgram
    - 成功时返回非0的无符号Handle值，指涉Shader程序对象）；
    -  OpenGL通过一个名为shaderProgram的对象来与shader交互
      - 也可以说shaders通过这个对象连接到我们的OpenGL应用程序中
        - 它具体地指涉shader过程。宏观概念上类似于我们平时写的“程序”
          - 不过它是基于GPU的；
  - 5把之前创建的shaders，Attach到这个shaderProgram对象上（glAttachShader
    - 当然理论上可以attach多于一个的同类型完整的shader，譬如vertexShader
    - 但是在一个特定的渲染流程中只允许其中一个起作用
  - 6链接shaderProgram    glLinkProgram
    - 应该以GL_LINK_STATUS为参调用glGetProgramiv检查链接是否成功。
    - 至此一个shader程式装载完毕。在进入一批渲染流程前（即渲染一组物件前）
  - 7启用这个shaderProgram对象（glUseProgram）
      - 它就会在这批渲染流程中起作用了
      - 渲染完后可以（也应该）调用glUseProgram(NULL)来关闭这种介入
      - 或者以其他shaderProgram渲染别的物件。
# attribute变量
    - 一般是指vertex attribute（顶点属性）—— 每个顶点都有一份
    - 在vertexShader中，我们处理的是每个顶点
      - 而我们希望传入的变量时每个顶点各异的时候  
      - 就使用这种变量（在shader中以attribute为限定符）
    - 它不必是传统意义上的“顶点属性”（顶点位置、法线之类）
      - 但它确实又是一种顶点的“属性”
    - 需要在GPU里的Shader的存储空间中有固定的位置
      - 在链接shaderProgram（见上文）之前，这个位置是未确定的
      - 因此我们可以在这个shaderProgram调用glLinkProgram之前
        - 为这个attribute变量指定一个位置（用无符号值表示）
        - glBindAttribLocation
      - 另一种获得这个“位置”的方法
        - 我们不要去显式设定这个位置，而是去获取它
        - 通常，如果shader里有attribute变量，且我们没有为它绑定一个位置
        - 那么shaderProgram在链接后，会自动为它分配一个位置
        - 我们可以在任何需要的时候获取（查询）这个位置：
          - glGetAttribLocation
          - 就不必局限于在shaderProgram的创建和链接之间去绑定了
        - 这里返回一个有符号的int值，因为当要查询的这个变量在shader中不存在
          - 或者它没有作用（非活动的：non-active），就会返回-1，否则才是它的位置
      - 直接在GLSL代码中指定这些位置值
        - layout(location = 0) in vec3 attrib_position;  

# uniform变量
    - uniform变量的特点是，对于一个vertexShader内处理的每个顶点（或者fragmentShader内处理的每个像素,它都是不变的
      - 事实上它相当于一个全局量，并非每个顶点/几何/像素所拥有的变量
      - （只是uniform对它们每一个都public而已）
    - 在一个渲染流程中改变它的值也是不可能的
    - 常在shaderProgram  LINK 后GET一个uniform变量的位置
      - 然后向这个位置传送数据（glGetUniformLocation/glUniform）
    - 只有在USE一个shaderProgram后，才能做这样的事情
    -  有些时候，给一个渲染对象类关联一个shader相关类的指针  
        - shaderProgram启用与否,完全交给这个渲染对象类
          - 还是得在上层为shader指定uniform数据
          - 这时候，可以在shader类指针之外，再关联一个map<uniform位置，uniform数据>
            - （当然了这个“数据”还得根据uniform变量类型来细分）
            - 我们只把数据传给这个map
            - 当shader在渲染对象类里头被启用之后
              - 立即就把这些数据都通过glUniform传送
            - glProgramUniform
                - 它比起往常的glUniform要多一个参数用来接收一个ShaderProgram的ID。在建立ShaderProgram后，我们也不需要glUseProgram来预先绑定它就可以直接取得某个uniform变量的location值并用glProgramUniform系列函数关联数据，而且这个数据在其后运行期间的每次glUseProgram后都不会失效。从理论上将，这族函数完全可以替代glUniform系列函数（是它们功能的一个超集），但是就不知道会不会有性能上的损失了（这个暂时目前找不到说法），所以我暂时建议是只对那些非动态变化的uniform变量使用了
# Uniform Buffer Object(UBO)
      - UBO，顾名思义，就是一个装载Uniform变量数据的Buffer Object。
      - 是显存中一块用于储存特定数据的区域了
      - 在OpenGL端，它的创建、更新、销毁的方式都与其他Buffer Object没什么区别
        - 我们只不过把一个或多个uniform数据交给它，以替代glUniform的方式传递数据而已
        - 这些数据是给到这个UBO，存储于这个UBO上，而不再是交给ShaderProgram，所以它们不会占用这个ShaderProgram自身的uniform存储空间
        - 所以UBO是一种全新的传递数据的方式，从路径到目的地，都跟传统uniform变量的方式不一样
        - 自然，对于这样的数据，在Shader中不能再使用上面代码中的方式来指涉了
        - 随着UBO的引入，GLSL也引入了uniform block这种指涉工具
# varying变量
      - varying变量主要用于在Shader Stage间进行传递
        - 注意的是在光栅化(Rasterization)的时候，这些变量也会跟着一起被光栅插值
        - 那如果我们不想某个顶点属性被光栅化，该怎么办呢？

        - 要让GLSL中某个作为顶点属性的varying变量不被光栅化
          - 只要在它前面加一个flat关键字就可以了
          - 这样它就像上述的那样，到达Fragmen Shader的图元上所有像素的该varying值都是相同的值（provoking vertex上的值）

#  fragmentShader输出
      - 一般来说，输出的是颜色值，输出目标是Frame Buffer
      - 这又包括常规的输出到屏幕Buffer
      - 输出到FBO（[学一学，FBO] ）
      - 另外还可以通过MRT（Multi Render Target）输出到两个以上的FBO中
      - 但是，这些对于Fragment Shader来说并没太多不一样：
        - 通过ShaderProgram Link前的 glBindFragDataLocation 指定输出到第几个Buffer（默认是0）
        - 类似于上述的attribute变量，我们也可以直接通过layout来指定这个location值
