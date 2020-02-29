# Chart 文件结构

chart 被组织为一个目录内的文件集合。目录名称是 chart 的名称（没有版本信息）。例如，描述 WordPress 的 chart 将被存储在 wordpress / 目录中。在这个目录里面，Helm 期望如下这样一个的结构的目录树：

```s
wordpress/
  Chart.yaml          # A YAML file containing information about the chart
  LICENSE             # OPTIONAL: A plain text file containing the license for the chart
  README.md           # OPTIONAL: A human-readable README file
  requirements.yaml   # OPTIONAL: A YAML file listing dependencies for the chart
  values.yaml         # The default configuration values for this chart
  charts/             # A directory containing any charts upon which this chart depends.
  templates/          # A directory of templates that, when combined with values,
                      # will generate valid Kubernetes manifest files.
  templates/NOTES.txt # OPTIONAL: A plain text file containing short usage notes
```

Helm 保留使用 charts / 和 templates / 目录以及上面列出的文件名称。其他文件将被忽略。

## Chart.yaml 文件

Chart.yaml 文件是 chart 所必需的。它包含以下字段：

```yml
apiVersion: The chart API version, always "v1" (required)
name: The name of the chart (required)
version: A SemVer 2 version (required)
kubeVersion: A SemVer range of compatible Kubernetes versions (optional)
description: A single-sentence description of this project (optional)
keywords:
  - A list of keywords about this project (optional)
home: The URL of this project's home page (optional)
sources:
  - A list of URLs to source code for this project (optional)
maintainers: # (optional)
  - name: The maintainer's name (required for each maintainer)
    email: The maintainer's email (optional for each maintainer)
    url: A URL for the maintainer (optional for each maintainer)
engine: gotpl # The name of the template engine (optional, defaults to gotpl)
icon: A URL to an SVG or PNG image to be used as an icon (optional).
appVersion: The version of the app that this contains (optional). This needn't be SemVer.
deprecated: Whether this chart is deprecated (optional, boolean)
tillerVersion: The version of Tiller that this chart requires. This should be expressed as a SemVer range: ">2.0.0" (optional)
```

如果熟悉 Chart.yaml Helm Classic 的文件格式，注意到指定依赖性的字段已被删除。这是因为新的 chart 使用 charts / 目录表示依赖关系。其他字段将被忽略。

## Charts 和版本控制

每个 chart 都必须有一个版本号。版本必须遵循 SemVer 2 标准。与 Helm Class 格式不同，Kubernetes Helm 使用版本号作为发布标记。存储库中的软件包由名称加版本识别。例如，nginx version 字段设置为 1.2.3 将被命名为：

```s
nginx-1.2.3.tgz
```

更复杂的 SemVer 2 命名也是支持的，例如 version: 1.2.3-alpha.1+ef365。但非 SemVer 命名是明确禁止的。虽然 Helm Classic 和 Deployment Manager 在 chart 方面都非常适合 GitHub，但 Kubernetes Helm 并不依赖或需要 GitHub 甚至 Git。因此，它不使用 Git SHA 进行版本控制。

许多 Helm 工具都使用 Chart.yaml 的 version 字段，其中包括 CLI 和 Tiller 服务。在生成包时，helm package 命令将使用它在 Chart.yaml 中的版本名作为包名。系统假定 chart 包名称中的版本号与 Chart.yaml 中的版本号相匹配。不符合这个情况会导致错误。

### appVersion 字段

请注意，appVersion 字段与 version 字段无关。这是一种指定应用程序版本的方法。例如，drupal chart 可能有一个 appVersion: 8.2.1，表示 chart 中包含的 Drupal 版本（默认情况下）是 8.2.1。该字段是信息标识，对 chart 版本没有影响。

### 弃用 charts

在管理 chart tepo 库中的 chart 时，有时需要弃用 chart。Chart.yaml 的 deprecated 字段可用于将 chart 标记为已弃用。如果存储库中最新版本的 chart 标记为已弃用，则整个 chart 被视为已弃用。chart 名称稍后可以通过发布未标记为已弃用的较新版本来重新使用。废弃 chart 的工作流程根据 helm/charts 项目的工作流程如下：

- 更新 chart 的 Chart.yaml 以将 chart 标记为启用，并且更新版本
- 在 chart Repository 中发布新的 chart 版本
- 从源代码库中删除 chart（例如 git）

## Chart 许可证文件，自述文件和说明文件

Chart 还可以包含描述 chart 的安装，配置，使用和许可证的文件。LICENSE 文件是一个纯文本文件，包含 chart 的 [许可证](https://en.wikipedia.org/wiki/Software_license)。 Chart 可以包含许可证，它可能在模板中具有编程逻辑，因此不仅仅是配置。 如果需要，还可以为 chart 安装的应用程序提供单独的许可证。

Chart 的自述文件应由 Markdown（README.md）语法格式化，并且通常应包含：

- chart 提供的应用程序或服务的描述
- 运行 chart 的任何前提条件或要求
- 选项 `values.yaml` 和默认值的说明
- 任何其他可能与安装或配置 chart 相关的信息

chart 还可以包含一个简短的纯文本 `templates/NOTES.txt` 文件，在安装后以及查看版本状态时将打印出来。此文件将作为模板 [template](https://whmzsu.github.io/helm-doc-zh-cn/chart/charts-zh_cn.html#templates-and-values) 进行评估 ，并可用于显示使用说明，后续步骤或任何其他与发布 chart 相关的信息。例如，可以提供用于连接到数据库或访问 Web UI 的指令。由于运行时，该文件被打印到标准输出 `helm install` 或 `helm status`，建议保持内容简短并把更多细节指向自述文件。
