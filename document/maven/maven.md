# 1. maven生命周期

## 1.1 概述

1. maven出现以前，项目构建的生命周期就已经存在了。软件开发人员每天都在对项目惊醒清理，编译，测试及部署，而且往往使用不同的方式做类似的工作。maven生命周期对所有的构建过程进行抽象和同意，总结了一套高度完善，易扩展的生命周期。包含了项目的清理，初始化，编译，测试，打包，集成测试，验证，部署，站点生成等几乎所有的构建步骤。

## 1.2 三套生命周期

1. maven拥有三套相互独立的生命周期，分别是clean(清理项目)，default(构建项目)和site(建立项目站点)。仅调用某套生命周期的某阶段不会对其他生命周期产生影响。每个生命周期都包含多个有序的阶段(phase)，并且后面的阶段依赖于千米那的阶段

### 1.2.1 Clean Lifecycle--目的是清理项目，包含三个阶段

1. pre-clean
   * 执行清理前需要完成的工作
2. clean
   * 清理上一次构建生成的所有文件
3. post-clean
   * 完成清理后需要完成的工作

### 1.2.2 Default Lifecycle--目的是构建项目，生命周期中最核心的部分

1. validate
   * 验证项目是否正确以及是否提供必要信息
2. initialize
   * 初始化构建状态，例如设置属性或创建目录
3. generate-source
   * 生成所有包含在编译中的源代码
4. process-sources
   * 处理项目主资源文件。对src/main/resources目录的内容进行变量替换等工作后，复制到项目输出的主classpath目录中
5. generate-resources
   * 生成包含在包中的资源
6. process-resources
   * 将资源复制并处理到目标目录中，为打包做准备
7. compile
   * 编译项目的主源码。编译src/main/resources目录下的java文件至项目输出的主classpath目录中
8. process-classes
   * 编译处理后的class文件
9. generate-test-sources
   * 生成所有包含在编译中的测试代码
10. process-test-sources处理项目测试员文件。对src/test/resources目录的内容进行变量替换等工作后，复制到项目输出的测试classpath目录中
11. generate-test-resources
    * 创建测试资源
12. process-test-resources
    * 将资源复制并处理到目标代码中
13. test-compile
    * 编译项目的测试代码。编译src/test/resources目录下的java文件至项目输出的测试classpath目录中
14. process-test-classes
    * 处理测试代码编译后的class文件
15. test
    * 使用单元测试框架运行测试。测试代码不会被打包或部署
16. prepare-package
    * 打包前的准备工作。比如解压缩，处理版本
17. package
    * 接受编译好的代码，打包成可发布的格式
18. pre-integration-test
    * 在集成测试执行之前所需的操作，比如设置环境等
19. integration-test
    * 如果需要，可将程序包处理并部署到可运行继承测试的环境中
20. post-integration-test
    * 完成继承测试后所需的操作。比如清理环境等
21. verity
    * 运行所有检查来验证包是否有效及是否符合质量标准
22. install
    * 将包安装到maven本地仓库，供本地其他maven项目使用
23. deploy
    * 将最终的包复制到远程仓库，供其他开发人员和maven项目使用

### 1.2.3 Site Lifecycle--目的是建立和发布项目站点，maven可以基于pom包含的信息，自动生成一个站点，方便团队交流和发布项目信息。该生命周期包含一下阶段

1. pre-site
   * 执行一些在生成项目站点之前需要完成的工作
2. site
   * 生成项目站点文档
3. post-site
   * 完成生成项目站点后需要完成的工作
4. siet-deploy
   * 将生成的项目站点发布到服务器上

# 2. maven插件

1. 在maven core中仅仅定义了抽象的生命周期，具体的实现是由插件完成的，而插件是以独立的构建形式存在

## 2.1 插件目标(Plugin Goal)

1. 一个插件往往能够完成多个功能，如果我们为其中每一个功能都实现一个独立的插件，显然不可取，因为这些功能别后有很多可以复用的代码。因此，这些功能都聚集在一个插件里，每个功能就是一个插件目标。例如maven-dependency-plugin由十几个目标，分析项目依赖dependency:analyze，列出项目的依赖树dependency:tree，列出项目所有已解析的依赖dependency:list。这是种通用写法，冒号前为插件前缀，冒号后为插件目标。

## 2.2 内置绑定

1. maven为了让用户几乎不需要任何配置就可以构建maven项目，在核心为一些主要的生命周期阶段绑定了插件的目标

### 2.2.1 clean与maven-clean-plugin:plugin

1. clean生命周期由pre-clean，clean，post-clean三个阶段。其中clean与maven-clean-plugin:clean绑定。maven-clean-plugin仅有clean这一目标，作用就是删除项目的输出目录

### 2.2.2 site与maven-site-plugin:site

1. site与maven-clean-site:plugin绑定用于生成项目站点。

### 2.2.3 site-deploy和maven-site-plugin:deploy

1. site-deploy和maven-site-plugin:deploy绑定，用于将项目站点部署到远程服务器上

## 2.3 maven常用插件整理

### 2.3.1 maven-compiler-plugin

1. 设置maven编译的jdk版本                                                                                                                                                                          