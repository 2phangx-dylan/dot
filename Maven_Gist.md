# 前言

> - 本篇描述了关于Maven的一些重要概念。

# Maven构建生命周期

- Maven构建生命周期定义了一个项目构建跟发布的过程。
- 一个典型的Maven构建（build）生命周期是由以下几个阶段的序列组成的。

![img](images/Maven_Gist.images/7642256-c967b2c1faeba9ce.png)

|       阶段       |   处理   |                           描述                           |
| :--------------: | :------: | :------------------------------------------------------: |
| 验证（validate） | 验证项目 |          验证项目是否正确且所有必须信息是可用的          |
| 编译（compile）  | 执行编译 |                  源代码编译在此阶段完成                  |
|   测试（test）   |   测试   |       使用适当的单元测试框架（例如junit）运行测试        |
| 包装（package）  |   打包   |           创建JAR/WAR包，如在pom.xml中提及的包           |
|  检查（verify）  |   检查   |         对集成测试的结果进行检查，以保证质量达标         |
| 安装（install）  |   安装   |        安装打包的项目到本地仓库，以供其他项目使用        |
|  部署（deploy）  |   部署   | 拷贝最终的工程包到远程仓库中，以共享给其他开发人员和工程 |

- 为了完成default生命周期，这些阶段（包括其他未在上面罗列的生命周期阶段）将被按顺序地执行。
- Maven有以下三个标准的生命周期：
  1. clean：项目清理的处理
  2. default（或build）：项目部署的处理
  3. site：项目站点文档创建的处理

## 1. 构建阶段由插件目标构成

- 一个插件目标代表一个特定的任务（比构建阶段更为精细），这有助于项目的构建和管理。这些目标可能被绑定到多个阶段或者无绑定。不绑定到任何构建阶段的目标，可以在构建生命周期之外，通过直接调用执行。这些目标的执行顺序取决于调用目标和构建阶段的顺序。
- 例如，以下命令：
  - clean和package是构建阶段，dependency:copy-dependencies是目标

```console
mvn clean dependency: copy-dependencies package
```

- 这里的clean阶段将会被首先执行，然后dependency:copy-dependencies目标将会被执行，最终package阶段被执行。

## 2. Clean生命周期

- 当我们执行mvn post-clean命令时，Maven调用clean生命周期，它包含以下阶段：
  - pre-clean：执行一些需要在clean之前完成的工作
  - clean：移除所有上一次构建生成的文件
  - post-clean：执行一些需要在clean之后立刻完成的工作
- mvn clean中的clean就是上述的clean，在一个生命周期中，运行某个阶段的时候，它之前的所有阶段都会被运行，也就是说，如果执行mvn clean将运行一些两个生命周期阶段：

```console
pre-clean, clean
```

- 如果我们运行mvn post-clean，则以下三个生命周期会被运行：

```console
pre-clean, clean, post-clean
```

## 3. Default(Build)生命周期

- 这是Maven的主要生命周期，被用于构建应用，包括以下23个阶段：

| 生命周期阶段                                | 描述                                                         |
| :------------------------------------------ | :----------------------------------------------------------- |
| validate（校验）                            | 校验项目是否正确并且所有必要的信息可以完成项目的构建过程。   |
| initialize（初始化）                        | 初始化构建状态，比如设置属性值。                             |
| generate-sources（生成源代码）              | 生成包含在编译阶段中的任何源代码。                           |
| process-sources（处理源代码）               | 处理源代码，比如说，过滤任意值。                             |
| generate-resources（生成资源文件）          | 生成将会包含在项目包中的资源文件。                           |
| process-resources （处理资源文件）          | 复制和处理资源到目标目录，为打包阶段最好准备。               |
| compile（编译）                             | 编译项目的源代码。                                           |
| process-classes（处理类文件）               | 处理编译生成的文件，比如说对Java class文件做字节码改善优化。 |
| generate-test-sources（生成测试源代码）     | 生成包含在编译阶段中的任何测试源代码。                       |
| process-test-sources（处理测试源代码）      | 处理测试源代码，比如说，过滤任意值。                         |
| generate-test-resources（生成测试资源文件） | 为测试创建资源文件。                                         |
| process-test-resources（处理测试资源文件）  | 复制和处理测试资源到目标目录。                               |
| test-compile（编译测试源码）                | 编译测试源代码到测试目标目录.                                |
| process-test-classes（处理测试类文件）      | 处理测试源码编译生成的文件。                                 |
| test（测试）                                | 使用合适的单元测试框架运行测试（Juint是其中之一）。          |
| prepare-package（准备打包）                 | 在实际打包之前，执行任何的必要的操作为打包做准备。           |
| package（打包）                             | 将编译后的代码打包成可分发格式的文件，比如JAR、WAR或者EAR文件。 |
| pre-integration-test（集成测试前）          | 在执行集成测试前进行必要的动作。比如说，搭建需要的环境。     |
| integration-test（集成测试）                | 处理和部署项目到可以运行集成测试环境中。                     |
| post-integration-test（集成测试后）         | 在执行集成测试完成后进行必要的动作。比如说，清理集成测试环境。 |
| verify （验证）                             | 运行任意的检查来验证项目包有效且达到质量标准。               |
| install（安装）                             | 安装项目包到本地仓库，这样项目包可以用作其他本地项目的依赖。 |
| deploy（部署）                              | 将最终的项目包复制到远程仓库中与其他开发者和项目共享。       |

> - 关于Maven生命周期相关的重要概念说明：
>   - 当一个阶段通过Maven命令调用时，例如mvn compile，只有该阶段之前以及包括该阶段在内的所有阶段会被执行。
>   - 不同的Maven目标将根据打包的类型（JAR/WAR/EAR），被绑定到不同的Maven生命周期阶段。

- 在开发环境中，通常使用以下命令去构建、安装工程到本地仓库

```console
mvn install
```

- 这个命令在执行install阶段前，会按顺序执行default生命周期的阶段（validate、compile、package等等），我们只需要调用最后一个阶段的命令即可。
- 在构建环境中，通常使用以下命令，纯净地构建和部署项目到共享仓库中

```console
mvn clean deploy
```

- 这行命令也可以用于多模块的情况下，即包含多个子项目的项目，Maven会在每一个子项目执行clean命令，然后再执行deploy命令

## 4. Site生命周期

- Maven Site插件一般用来创建新的报告文档、部署站点等。
  - pre-site：执行一些需要在生成站点文档之前完成的工作
  - site：生成项目的站点文档
  - post-site：执行一些需要在生成站点文档之后完成的工作，并且为部署做准备
  - site-deploy：将生成的站点文档部署到特定的服务器上