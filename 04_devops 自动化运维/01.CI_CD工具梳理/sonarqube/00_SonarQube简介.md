## SonarQube 简介

SonarQube 是一款本地代码分析工具，旨在检测30 多种编程语言、框架和 ` IaC platforms`平台中的编码问题。通过直接与您的CI 流水线或我们支持的 DevOps 平台集成，您的代码将根据一组广泛的规则进行检查，这些规则涵盖代码的许多属性，例如每个合并/拉取请求的可维护性、可靠性和安全问题

SonarQube 主要在各种规则分析下，保障你交付一份高质量干净的标准代码



### 如何清洁代码

​	**干净的代码**是所有代码的标准，可产生安全、可靠和可维护的软件，因此，编写干净的代码对于维护健康的代码库至关重要。这适用于所有代码：源代码、测试代码、基础设施即代码、粘合代码、脚本等等

​	Sonar 的[Clean as You Code](https://docs.sonarsource.com/sonarqube/latest/user-guide/clean-as-you-code/)方法是一种软件开发实践，其原则是新代码（您最近添加或修改的代码）需要符合质量标准。 Sonar 解决方案通过在新代码中检测到问题时向您发出警告来实现“代码清洁”，通过逐步改进整个代码库来帮助您保持高标准并专注于代码质量

​	SonarQube 附带专为每种支持的语言设计的内置**质量配置文件**，称为**Sonar Way**配置文件。 **Sonar 方式**激活了一组适用于大多数项目的规则，并且是帮助您在组织中实施干净代码实践的起点

### SonarQube 解决方案

Sonar 产品旨在帮助实现清洁代码状态。通过链接[SonarLint](https://docs.sonarsource.com/sonarlint/)使用[SonarCloud](https://docs.sonarsource.com/sonarcloud/)或 SonarQube，可以在开发过程的每个阶段执行检查；我们称之为***SonarQube解决方案***。这意味着您的项目设置、新代码定义和质量配置文件将在本地应用到 IDE 中的分析。 Sonar 解决方案旨在帮助您实现干净代码的状态。在 SonarCloud 中管理的项目设置、新代码定义和质量配置文件将在本地应用到 IDE 中的分析

- SonarLint 有助于及早发现问题，因为它在 IDE 中提供即时反馈。当工程师编写代码时，他们甚至可以在提交之前发现并修复问题
-  [SonarQube](https://docs.sonarsource.com/sonarqube/latest/analyzing-source-code/pull-request-analysis/introduction/)和[SonarCloud](https://docs.sonarsource.com/sonarcloud/improving/pull-request-analysis/)将在合并之前分析每个拉取请求，从而针对代码问题提供另一层保护
- [SonarQube](https://docs.sonarsource.com/sonarqube/latest/user-guide/quality-gates/)和[SonarCloud](https://docs.sonarsource.com/sonarcloud/improving/quality-gates/)分析每个构建上的代码，作为 CI/CD 工作流程的一部分，并结合[质量配置文件](https://docs.sonarsource.com/sonarqube/latest/instance-administration/quality-profiles/)，防止有问题的代码发布到生产环境

Sonar 解决方案可帮助工程师关注新代码，从而帮助您整合[“Clean as You Code”](https://docs.sonarsource.com/sonarqube/latest/user-guide/clean-as-you-code/)方法。在开发过程中专注于编写新的、干净的代码可确保发布到生产环境的所有代码都将随着时间的推移逐步得到改进



### SonarQube Connected Mode

​	Connected Mode将 SonarCloud 与 SonarLint 结合起来，提供完整的 Sonar 解决方案。在Connected Mode下，当质量门发生更改或向用户分配新问题时，SonarCloud 会向 SonarLint 发送通知。创建或编辑连接设置时，可以从 SonarLint UI 启用或禁用智能通知。此外，SonarLint 通过使用服务器上定义的新代码定义，帮助工程师专注于编写干净的代码

### 更多入门资源

- [Server installation and setup 服务器安装和设置](https://docs.sonarsource.com/sonarqube/latest/setup-and-upgrade/overview/)
- [How to create and import projects 如何创建和导入项目](https://docs.sonarsource.com/sonarqube/latest/project-administration/creating-and-importing-projects/)
- [How to administer quality profiles 如何管理质量档案](https://docs.sonarsource.com/sonarqube/latest/instance-administration/quality-profiles/)
- [How to set up portfolios 如何建立投资组合](https://docs.sonarsource.com/sonarqube/latest/project-administration/managing-portfolios/)

