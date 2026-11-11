# Maven 构建与环境规范

适用于使用 Maven 构建并运行的 Java 应用。页面资源刷新要求适用于由 Java 应用提供模板或静态资源的 Web 项目。环境的 YAML 命名和配置分离沿用[通用环境配置](<10-10 通用开发规范.md#environment-yaml>)。

<a id="maven-environment-profiles"></a>
## 1. 环境 Profile

使用 Maven 构建并运行的应用必须在 POM 中定义可用的 `dev` Profile。可以预留 `test`、`inte`、`prod` Profile 及对应的 YAML，配置内容可先为空。各环境的用途和启用条件见[通用环境配置](<10-10 通用开发规范.md#environment-yaml>)。

- **一次选择一个环境。** Maven 环境 Profile 明确对应运行环境及配置位置。例如 `-Pdev` 对应开发环境的 `application-dev.yml`。按环境目录组织时，写清 Profile 对应哪个目录。安全扫描等独立构建 Profile 可以与环境 Profile 同时使用。
- **本地开发可以默认 dev。** 打开发包时显式执行 `mvn -Pdev package`。提供 Maven Wrapper 的项目使用 `./mvnw -Pdev package`，Windows PowerShell 下使用 `.mvnw.cmd -Pdev package`。针对具体环境的持续集成和发布任务应明确指定目标。选了多个环境，或指定的环境尚未启用时，应报错。
- **构建与运行的环境选择一致。** 制品已在构建时选定环境的，IDE、脚本和直接运行的入口都沿用该环境，显式指定的环境与制品冲突时报错。同一制品跨环境部署时，由部署入口显式选定目标环境并核对实际生效结果。使用 Spring Boot 时，说明 Maven Profile 对应的 Spring Profile。
- **自动化测试单独准备运行条件。** `-Ptest` 选择可部署的测试环境，`mvn test` 执行构建中的测试阶段。自动化测试按自己的约定准备测试数据和依赖服务。
- **开发数据库的选择归 dev 配置。** 共享库和个人本地库默认沿用 `dev` Profile。需要分别打包开发配置时，可以使用明确的构建变体，两种方式仍属于同一开发环境。连接管理见[开发数据库](<10-10 通用开发规范.md#development-database>)。

<a id="pom-readability"></a>
### 1.1 POM 的可读性

POM 沿用[通用配置组织规则](<10-10 通用开发规范.md#readable-project-structure>)。阅读一个环境 Profile 时，应能顺着定义找到它改变的配置和生效位置。例如，多个环境互斥时，校验条件要能直接看出允许的选择，错误提示要说明冲突和修正办法。

<a id="maven-packaged-configuration"></a>
### 1.2 配置打包与验证

- **明确复制和打包范围。** 按交付方式声明哪些配置进入编译输出和最终制品。构建产物绑定单个环境时，只打包该环境和共用的资源。私有配置与样例同目录时，按文件范围选择允许发布的配置，并在最终制品中核对敏感文件、仅供参考的样例和旧输出均未混入。
- **按真实入口核对结果。** 检查实际资源输出及最终 JAR 或 WAR，使用嵌套配置包时也检查其内容。调整配置打包规则时，按环境打包且支持增量构建的项目，验证切换环境后重新打包，确认旧环境资源没有残留。通过实际启动入口核对生效的配置。验证文件排除规则时可使用模拟私有文件。涉及路径或入口迁移时，沿用[通用迁移要求](<10-10 通用开发规范.md#configuration-migration>)。

<a id="maven-dev-resources"></a>
## 2. dev 开发时自动更新

`dev` 的开发启动配置应减少每次修改后的手工操作。启动一次后，保存源码即可继续调试。项目应提供明确的启动命令或可复用的 IDE 配置。

- 修改 HTML 模板、CSS 或 JavaScript 后，**保存文件并刷新浏览器即可看到新内容**，无需重新打包或重启应用。浏览器自动刷新可以作为开发便利功能。
- 开发启动入口优先直接读取实际的模板和静态资源源码目录。采用 Thymeleaf 时，关闭 dev 的模板缓存，并配置静态资源的开发缓存策略，使下一次页面请求可以取到最新资源。
- 页面资源在独立 Maven 子模块时，开发启动入口要显式接入该模块的资源目录。Spring Boot Maven 插件的 `addResources` 作用于执行插件的模块。启动模块与页面模块分开时，需另行配置页面资源路径。
- 修改 Controller 等 Java 代码后，开发流程应在保存后自动编译，并在编译成功后自动加载改动，无需手工打包、停止或重新启动服务。修改方法实现可使用调试器热替换。新增接口、修改路由、方法签名或组件装配时，开发工具应自动重新加载应用。Spring Boot 项目可用 IDE 自动编译配合 [DevTools 自动重启](https://docs.spring.io/spring-boot/reference/using/devtools.html#using.devtools.restart)。

接入时用项目正式提供的 dev 命令或 IDE 配置启动一次，按下列方式验收。

- 页面资源。改一处 HTML 文案和一处 CSS 样式，保存并刷新，确认页面已变且应用进程没有重启。提供 JS 的项目再验证一处脚本变更。
- Controller。保存一处已有方法的返回值修改，再保存一处路由变更。分别通过 HTTP 验证新行为，期间无需手工编译、打包或重启。
- 集成和生产。确认实际加载的是随包发布的资源，开发时的自动更新只在 dev 开发入口启用。

<a id="maven-lombok"></a>
## 3. Lombok 编译配置

- 在用到 Lombok 的模块声明编译期依赖。Maven 使用 `provided`，由父 POM 或 BOM 统一管理版本。编译器配置明确声明 Lombok 注解处理器，命令行和 IDE 编译使用同一份模块配置。
- 验证实际 Maven 编译是否生成所需方法。涉及打包时检查最终 JAR 或 WAR。Spring Boot 重打包可能纳入 `provided` 依赖，使用相应排除项，确保 Lombok 留在编译期。
- 源码中的注解用法见 [Java 的 Lombok 约定](<30-10 Java 通用代码规范.md#java-lombok>)。配置细节见 [Lombok Maven 说明](https://projectlombok.org/setup/maven)及 [Spring Boot 打包说明](https://docs.spring.io/spring-boot/maven-plugin/packaging.html)。

参考 [Maven Build Profiles](https://maven.apache.org/guides/introduction/introduction-to-profiles.html)、[Spring Boot Profiles](https://docs.spring.io/spring-boot/reference/features/profiles.html) 和 [Spring Boot Maven 插件运行选项](https://docs.spring.io/spring-boot/maven-plugin/run.html)。
