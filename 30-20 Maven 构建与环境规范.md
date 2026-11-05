# Maven 构建与环境规范

适用于使用 Maven 构建并运行的 Java 应用。页面资源刷新要求适用于由 Java 应用提供模板或静态资源的 Web 项目。环境的 YAML 命名和配置分离沿用[通用环境配置](<10-10 通用开发规范.md#environment-yaml>)。

<a id="maven-environment-profiles"></a>
## 1. 环境 Profile

在 `pom.xml` 中提供 `dev`、`inte`、`prod` 三个环境 Profile。每次选择一个环境 Profile。安全扫描等独立的构建 Profile 可以与环境 Profile 一起使用。

| Maven 选项 | 用途 | 运行要求 |
| --- | --- | --- |
| `-Pdev` | 本地开发与调试 | 使用开发环境配置。提供支持资源刷新和代码自动重载的开发启动入口。 |
| `-Pinte` | 集成测试与联调 | 使用独立的集成环境配置，通过打包后的资源验证应用。 |
| `-Pprod` | 生产发布 | 部署时显式选择生产运行环境，使用已验收的制品和生产配置。生产环境关闭源码目录读取和开发热重载。 |

**打开发包时显式选用 `dev` Profile**，执行 `mvn -Pdev package`。项目提供 Maven Wrapper 时，使用 `./mvnw -Pdev package`，Windows PowerShell 下使用 `.\mvnw.cmd -Pdev package`。

Maven 的 `-P` 选择构建配置。通过 Maven 启动应用时，按[通用环境配置](<10-10 通用开发规范.md#environment-yaml>)传入同名运行环境。以 Spring Boot 为例，启动入口将选中的 Maven 环境 Profile 映射到同名 Spring Profile。运行已打包的 JAR 时，通过 `SPRING_PROFILES_ACTIVE` 或 `--spring.profiles.active` 选择环境。运行所需的环境配置缺失时应明确报错。集成验收和生产优先使用同一构建产物，环境差异由启动配置注入。

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

参考 [Maven Build Profiles](https://maven.apache.org/guides/introduction/introduction-to-profiles.html)、[Spring Boot Profiles](https://docs.spring.io/spring-boot/reference/features/profiles.html) 和 [Spring Boot Maven 插件运行选项](https://docs.spring.io/spring-boot/maven-plugin/run.html)。
