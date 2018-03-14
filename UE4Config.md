# Config
- 配置文件（Config）其实就是.ini文件
- 用于设置加载时要初始化的属性的值
- 配置信息按照键值对的格式来实现
- 配置文件一般只存在于以下四个路径
  - \Engine\Config及其子目录
  - \Engine\Saved\Config及其子目录（引擎运行后生成）
  - Projects\[ProjectName]\Config及其子目录
  - Projects\[ProjectName]\Saved\Config及其子目录（游戏项目运行后生成）

# Section
- 个配置文件里面有很多模块，每个模块的标题就是一个section
# Flush
- 类FConfigCacheIni有一个名为Flush()的方法
  - 它的表面意思是冲洗，奔涌，在代码里面表示将内存信息（即缓存的配置信息）准确无误的书写到文件里面
# SaveConfig与LoadConfig
- SaveConfig函数：这个函数用来将配置信息保存到GConfig并保存到文件里面。（
  - SaveConfig函数是UObject的一个方法，这就表示所有继承于UObject的类都可以使用它
  - 根据图我们看到该函数有三个参数，第三个参数默认是GConfig（后面会讲到这个全局变量），一般不做修改
  - 第一个参数默认是CPF_Config，我们也可以指定为CPF_GlobalConfig
    - 表示我们要保存的是标记Config的属性还是标记Global的属性
  - 第二个参数要重点强调，他表示带有路径的文件名称
    - 如果我们不写该参数，那么SaveConfig就会找到当前调用他的类的UCLASS(Config=XXX)所标记的这个XXX文件，并将信息保存到这个文件
    - 如果我们给出了第二个参数，他就将信息保存到对应参数的文件里面
- LoadConfig函数：这个函数用来读取配置文件的信息并将该信息赋值给当前类对应的属性。（后面会进一步解释）
  - LoadConfig的功能是从配置文件里面读取特定的属性值并赋给指定类的对应属性
    - 第一个参数指定了要把值赋给哪个类的属性，第二个参数表示到哪个配置文件找对应的配置信息，第三个参数表示是哪种标记（Config还是GlobalConfig），第四个参数表示指定要赋值的属性。
    - 如果不指定第4个参数，就会读取配置并给所有标记UProperty(Config)的属性赋值

- UCLASS(config=FileName)：表示这个类默认条件下将配置信息保存到哪个配置文件，config后面的文件名可以是任意字符串。
- UCLASS(perObjectConfig)：表示这个类的配置信息将会基于每个实例进行存储。
- UCLASS(config=XXX,configdonotcheckdefaults)：表示这个类对应的配置文件不会检查XXX层级上层的DefaultXXX配置文件是否有该信息（后面会解释层级），就直接存储到Saved目录下。
- UPROPERTY(config,globalconfig)：不指定Section的情况下，标记config的这个属性在保存到配置文件里面的时候会保存在当前类对应的Section部分。同理，加载的时候也会从当前类对应的Section下加载。

# 存储原理 GConfig
-配置文件的存储通过一个全局的变量GConfig来实现，GConfig是FConfigIni（配置信息缓存类）的一个实例化对象
- 定义如下FConfigCacheIni* GConfig =NULL;
- 在不同的情况下存储不同的信息

# 自动配置
- 指出应该从哪个配置文件中读取哪个变量，那么在包含这些变量的类的UCLASS
  - 宏中应赋予Config标识符
  - 必须为 Config 标识符提供类别（比如Game（游戏））
    - 这确定了从哪个配置文件中读取类的变量及将其保存到哪个配置文件中
    - 所有可能的分类都在类ConfigCacheIni（实际是GConfig对象）中进行定义
    - 所有配置文件的分类列表在第二部分里面已经描述
- 要想指定读取和保存到配置文件中的某个变量，也必须为该属性UPROPERTY()宏提供Config标识符
- 引擎执行SaveConfig的时候会遍历一遍所有的标记Config的属性
  - 判断当前对象的config属性是否与同属一类CDO的相同属性的值相同，如果相同证明没有任何修改，也就没有必要存储到配置文件里面去
- 所以有两种办法修改属性的值
  - 一是在一个方法里面修改该config属性的值，然后在游戏开始的时候调用
  - 另一种是将这个config属性设置为蓝图可读写，在蓝图配置里面修改
  - 最后，在这些修改操作执行之后一定要调用SaveConfig来保存配置信息（根据3.1的讲解，这里的SaveConfig我们一般不填第二个参数）
    - 同时就会将信息写到配置文件里面。如果上面的工作你都做了，但是没有执行SaveConfig是不会有任何效果的
# 手动编码配置
- 可以通过硬编码让配置信息保存到任意的文件里面。这里我们还是必须要用到SaveConfig函数，不过要设置好正确的参数。前面介绍过，SaveConfig()函数是UObejct方法，可以在任意继承于UObject并使用配置类修饰符的类上进行调用
- 只要是借助UProperty并且调用由SaveConfig()实现自动保存的，保存的变量就会位于按照格式 [(package).(classname)] 命名的Section中
- 下面举例在GameMode中如何使用手动编码将信息保存到指定的位置
  - 获取到全局的GConfig然后执行GConfig->SetBool(SectionStr,OptionStr,Value, GGameUserSettings); 操作，将该bool变量写到GConfig里面。当然引擎还提供了Set String，Integer，float等类型的配置变量
  - 然后在GameMode初始化阶段，或者其他合适的位置执行步骤1并调用SaveConfig保存。
    - GGameUserSettingsIni是全局的FString，保存的是GameUserSetting配置文件的路径，所以这里我们就可以把GameMode的配置信息保存到GameUserSetting里面了。其他的配置文件路径对于的变量可以在Core.cpp里面搜索到
- GGameUserSettingsIni是全局的FString，保存的是最高层级的GameUserSetting配置文件的路径。其他的配置文件路径对应的变量可以在Core.cpp里面搜索到。引擎在运行的时候会自动的帮我们给这些路径赋值

# 读取自定义文件并建立对应的层次结构
- 可以在合适的位置（GameInstance，GameMode，或者引擎的InitializeConfigSystem）调用FConfigCacheIni::LoadGlobalIniFile(GConfigIni, TEXT("Config"), NULL, NULL, true);
  - 之后代码就会给“Config”名称的文件建立一个层次结构，并在Saved/Config生成一个Config.ini文件。这里的GConfigIni是我们定义的全局路径，方便我们随时获取这个Config文件的位置
# 配置文件的存储流程看图
# 原理与特性分析
