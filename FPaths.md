## FPaths
- 检索游戏目录，引擎目录等路径助手
- FPaths::NormalizeDirectoryName
- FPaths::ConvertRelativePathToFull

- GetWrappedLaunchDir()
  - 启动时的工作目录，因为马上要把工作目录改为下面所说的exe所在目录，所以会先把当前的缓存起来
- FPlatformProcess::BaseDir()
  - 这个是最基本的，就是当前exe文件所在目录。
  - 也是最早被计算的一个目录，因为是所有依赖项的根结点，凡是要把相对路径转成全路径的，都是基于这个目录
  - 可以用【-BaseFromWorkingDir】来把BaseDir指向当前工作目录

- FPaths::EngineDir()
  - 引擎目录，用来定位很多擎内置资源。
  - 默认的值是../../../Engine/，而基准目录就是上面的BaseDir
  - 因为在发布后的目录结构里，游戏目录和引擎目录是平级的顶层子目录，exe文件会放在【游戏目录/Binaries/Win64/】下面，退三层后刚好找到Engine
  - 但是，这个目录可以用【GForeignEngineDir】来重载，前提是按默认的方法没找到引擎目录（在引擎目录下没有Binaries子目录）
  - 同时在GForeignEngineDir下有Binaries
  - 这也是开发阶段的默认配置，因为此时游戏还未打包，游戏目录与引擎目录不在一起，这时就只能通过GForeignEngineDir来定位引擎了
- GForeignEngineDir的实际值是什么呢？其实它是通过编译时传入的宏来定义的：

#if PLATFORM_DESKTOP
    #ifdef UE_ENGINE_DIRECTORY
        #define IMPLEMENT_FOREIGN_ENGINE_DIR() const TCHAR * GForeignEngineDir = TEXT( PREPROCESSOR_TO_STRING(UE_ENGINE_DIRECTORY) );
    #else
        #define IMPLEMENT_FOREIGN_ENGINE_DIR() const TCHAR * GForeignEngineDir = nullptr;
    #endif
#else
    #define IMPLEMENT_FOREIGN_ENGINE_DIR()
#endif

- 其中UE_ENGINE_DIRECTORY在UEBuildTarget.cs里有设定
  - string EnginePath = Utils.CleanDirectorySeparators(Utils.MakePathRelativeTo(ProjectFileGenerator.EngineRelativePath, Path.GetDirectoryName(OutputFilePath)), '/');
                    if (EnginePath.EndsWith("/") == false)
                    {
                        EnginePath += "/";
                    }
                    GlobalCompileEnvironment.Config.Definitions.Add("UE_ENGINE_DIRECTORY=" + EnginePath);
- 而IMPLEMENT_FOREIGN_ENGINE_DIR这个宏的调用出现在UE4Game.cpp里

#if IS_MONOLITHIC
PER_MODULE_BOILERPLATE
bool GIsGameAgnosticExe = true;
TCHAR GInternalGameName[64] = TEXT("");
IMPLEMENT_DEBUGGAME()
IMPLEMENT_FOREIGN_ENGINE_DIR()
#endif

- FPaths::RootDir()
  - 根目录，但其实是从引擎目录反推出来的，也就是找到【/Engine】这一段去掉后上一层
  - 但是如果引擎目录里没有/Engine这一段怎么办（上面说过可以重载为一个自定义的）？

- FPaths::GameDir()
  - 游戏目录，默认是与引擎目录同级的，以游戏名命名的目录，但是也可以通过【OverrideGameDir】重载
  - 本来是很简单，可是看代码推导过程极其复杂，而且结果也好难理解，明明已经得到最精简的绝对路径了，可最后返回的竟然还是一个充满../的相对路径，还绕来绕去好几层

- FPaths::GameContentDir returned L"../../../UnrealEngine/../hz413/Content/" FString &
- GameContentDir、GameConfigDir、GameSavedDir、GameIntermediateDir
