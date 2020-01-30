# 模板语法

# 对象

对象从模板引擎传递到模板中。而且您的代码可以传递对象（在查看 with 和 range 语句时，我们将看到示例）。甚至有几种方法可以在模板中创建新对象，例如稍后将介绍的元组功能。对象可以很简单，只有一个值。或者它们可以包含其他对象或功能。例如。Release 对象包含多个对象（例如 Release.Name），而 Files 对象具有一些功能。

## 内置对象

- `Release`：这个对象描述了 release 本身。它里面有几个对象：

  - `Release.Name`：release 名称，即是 `helm install --name` 命令中指定的名称。
  - `Release.Time`：release 的时间
  - `Release.Namespace`：release 的 namespace（如果清单未覆盖）
  - `Release.Service`：release 服务的名称（始终是 `Tiller`）。
  - `Release.Revision`：此 release 的修订版本号。它从 1 开始，每 `helm upgrade` 一次增加一个。
  - `Release.IsUpgrade`：如果当前操作是升级或回滚，则将其设置为 `true`。
  - `Release.IsInstall`：如果当前操作是安装，则设置为 `true`。

- `Values`：从 `values.yaml` 文件和用户提供的文件传入模板的值。默认情况下，Values 是空的。

- `Chart`：`Chart.yaml` 文件的内容。任何数据 Chart.yaml 将在这里访问。例如 {{.Chart.Name}}-{{.Chart.Version}} 将打印出来 mychart-0.1.0。chart 指南中 [Charts Guide](https://github.com/kubernetes/helm/blob/master/docs/charts.md#the-chartyaml-file) 列出了可用字段

- `Files`：这提供对 chart 中所有非特殊文件的访问。虽然无法使用它来访问模板，但可以使用它来访问 chart 中的其他文件。请参阅 "访问文件" 部分。

  - `Files.Get` 是一个按名称获取文件的函数（`.Files.Get config.ini`）
  - `Files.GetBytes` 是将文件内容作为字节数组而不是字符串获取的函数。这对于像图片这样的东西很有用。

- `Capabilities`：这提供了关于 Kubernetes 集群支持的功能的信息。

  - `Capabilities.APIVersions` 是一组版本信息。
  - `Capabilities.APIVersions.Has $version` 指示是否在群集上启用版本（`batch/v1`）。
  - `Capabilities.KubeVersion` 提供了查找 Kubernetes 版本的方法。它具有以下值：Major，Minor，GitVersion，GitCommit，GitTreeState，BuildDate，GoVersion，Compiler，和 Platform。
  - `Capabilities.TillerVersion` 提供了查找 Tiller 版本的方法。它具有以下值：SemVer，GitCommit，和 GitTreeState。

- `Template`：包含有关正在执行的当前模板的信息

- `Name`：到当前模板的 namespace 文件路径（例如 `mychart/templates/mytemplate.yaml`）

- `BasePath`：当前 chart 模板目录的 namespace 路径（例如 mychart/templates）。

# Values

Helm 模板提供的内置对象。四个内置对象之一是 Values，该对象提供对传入 Chart 的值的访问。其内容来自四个来源：

- chart 中的 `values.yaml` 文件
- 如果这是一个子 chart，来自父 chart 的 `values.yaml` 文件
- value 文件通过 helm install 或 helm upgrade 的 - f 标志传入文件（`helm install -f myvals.yaml ./mychart`）
- 通过 `--set`（例如 `helm install --set foo=bar ./mychart`）

上面的列表按照特定的顺序排列：values.yaml 在默认情况下，父级 chart 的可以覆盖该默认级别，而该 chart values.yaml 又可以被用户提供的 values 文件覆盖，而该文件又可以被 --set 参数覆盖。值文件是纯 YAML 文件。我们编辑 mychart/values.yaml，然后来编辑我们的 ConfigMap 模板。

## Values 值使用

删除默认带的 values.yaml，我们只设置一个参数：

```yaml
favoriteDrink: coffee
```

现在我们可以在模板中使用这个：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{.Release.Name}}-configmap
data:
  myvalue: "Hello World"
  drink: {{.Values.favoriteDrink}}
```

注意我们在最后一行 {{ .Values.favoriteDrink}} 获取 `favoriteDrink` 的值。

让我们看看这是如何渲染的。

```bash
$ helm install --dry-run --debug ./mychart
SERVER: "localhost:44134"
CHART PATH: /Users/mattbutcher/Code/Go/src/k8s.io/helm/_scratch/mychart
NAME:   geared-marsupi
TARGET NAMESPACE:   default
CHART:  mychart 0.1.0
MANIFEST:
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: geared-marsupi-configmap
data:
  myvalue: "Hello World"
  drink: coffee
```

由于 `favoriteDrink` 在默认 `values.yaml` 文件中设置为 `coffee`，这就是模板中显示的值。我们可以轻松地在我们的 helm install 命令中通过加一个 `--set` 添标志来覆盖：

```bash
helm install --dry-run --debug --set favoriteDrink=slurm ./mychart
SERVER: "localhost:44134"
CHART PATH: /Users/mattbutcher/Code/Go/src/k8s.io/helm/_scratch/mychart
NAME:   solid-vulture
TARGET NAMESPACE:   default
CHART:  mychart 0.1.0
MANIFEST:
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: solid-vulture-configmap
data:
  myvalue: "Hello World"
  drink: slurm
```

由于 `--set` 比默认 `values.yaml` 文件具有更高的优先级，我们的模板生成 `drink: slurm`。

values 文件也可以包含更多结构化内容。例如，我们在 values.yaml 文件中可以创建 `favorite` 部分，然后在其中添加几个键：

```yaml
favorite:
  drink: coffee
  food: pizza
```

现在我们稍微修改模板：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{.Release.Name}}-configmap
data:
  myvalue: "Hello World"
  drink: {{.Values.favorite.drink}}
  food: {{.Values.favorite.food}}
```

虽然以这种方式构建数据是可以的，但建议保持 value 树浅一些，平一些。当我们看看为子 chart 分配值时，我们将看到如何使用树结构来命名值。

## 删除默认的 Key

如果您需要从默认值中删除一个键，可以覆盖该键的值为 null，在这种情况下，Helm 将从覆盖值合并中删除该键。例如，stable 版本的 Drupal chart 允许配置 liveness 探测器，如果你配置自定义的 image。以下是默认值：

```yaml
livenessProbe:
  httpGet:
    path: /user/login
    port: http
  initialDelaySeconds: 120
```

如果尝试覆盖 liveness Probe 处理程序 `exec` 而不是 `httpGet`，使用 `--set livenessProbe.exec.command=[cat,docroot/CHANGELOG.txt]`，Helm 会将默认和重写的键合并在一起，从而产生以下 YAML：

```yaml
livenessProbe:
  httpGet:
    path: /user/login
    port: http
  exec:
    command:
      - cat
      - docroot/CHANGELOG.txt
  initialDelaySeconds: 120
```

但是，Kubernetes 会报错，因为无法声明多个 liveness Probe 处理程序。为了克服这个问题，你可以指示 Helm 过将 livenessProbe.httpGet 通设置为空来删除它：

```bash
helm install stable/drupal --set image=my-registry/drupal:0.1.0 --set livenessProbe.exec.command=[cat,docroot/CHANGELOG.txt] --set livenessProbe.httpGet=null
```

到这里，我们已经看到了几个内置对象，并用它们将信息注入到模板中。
