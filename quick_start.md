
## 快速开始

> 注意：从8.0版本开始，GitLab持续集成(CI)完全融合到GitLab本身，并且默认在所有项目下开启

## GitLab CI工作原理如下：

GitLab提供持续集成服务。如果在根目录添加 `.gitlab-ci.yml` 文件并且配置项目使用 `Runner`，那么每次`merge request`或`push`操作将触发一次构建。

`.gitlab-ci.yml` 文件告诉GitLab `runner`应该做什么。默认情况下它运行三个阶段：构建、测试和部署

`push`或`merge request`操作，如果一切运行正常（没有非零返回值），你会得到一个绿色的复选标记。这会很容易看到合并请求是否会导致测试失败。

大多数项目只使用GitLab CI服务运行测试套件，这样如果开发者破坏了一些东西就可以获得即时反馈。

简言之，CI的工作步骤可以终结为：
1. 在仓库的根目录添加 `.gitlab-ci.yml`文件
2. 配置 `Runner`

这时每次`push` 到Git仓库，`Runner`会自动编译，并且会在项目的`/builds`页面显示。

本文档假设你：
* 工作在8.0或更高版本的GitLab版本或使用[GitLab.com](https://gitlab.com/)
* 项目想使用GitLab CI

让我们去分解步骤去解决 GitLab CI问题。

## 创建 `.gitlab-ci.yml` 文件

在你创建文件之前，我们先简要解释下。

### `.gitlab-ci.yml` 是什么

`.gitlab-ci.yml`是配置CI将为你的项目做什么。它放置在下项目的根目录。

任何一个`push`操作，GitLab会寻找`.gitlab-ci.yml`文件并且根据文件的内容开始编译。

因为`.gitlab-ci.yml`文件存储在仓库中，版本版本控制的，老版本依然可以编译成功，`fork`也可以容易使用CI，分支会可以分开编译。可以在[blog](https://about.gitlab.com/2015/05/06/why-were-replacing-gitlab-ci-jobs-with-gitlab-ci-dot-yml/)中阅读使用`.gitlab-ci.yml`原因。

**注意**：`.gitlab-ci.yml`是[YAML](https://en.wikipedia.org/wiki/YAML)文件，所以你必须额外注意缩进，总是使用 `spaces`， 而不是`tab`。

### 创建简单的`.gitlab-ci.yml`文件

你需要在根目录创建一个名为`.gitlab-ci.yml`文件。下面是一个`Ruby on Rails`项目。

```
before_script:
  - apt-get update -qq && apt-get install -y -qq sqlite3 libsqlite3-dev nodejs
  - ruby -v
  - which ruby
  - gem install bundler --no-ri --no-rdoc
  - bundle install --jobs $(nproc)  "${FLAGS[@]}"

rspec:
  script:
    - bundle exec rspec

rubocop:
  script:
    - bundle exec rubocop

```

这是一个大多数`Ruby`应用程序的简单配置：
1. 定义两个jobs `rspec` 和 `rubocop` (名字是任意的)用于不同的命令执行。
2. 在每一个`job`之前，`before_script`命令将会执行

`.gitlab-ci.yml`文件约束`jobs`运行的时间和工作方式。`job`定义为带名字的顶级元素（在我们的例子中如：`rspec`和`rubocop`）并且总是包含`script`关键字。`Jobs`用来创建构建编译，选择`Runners`然后在`Runner`环境中执行。

重要的是每一个`job`都是相互独立运行的。

如果你想检查`.gitlab-ci.yml`文件是否有效，GitLab 实例有一个检查工具`/ci/lint`页面。或者在项目中的`Settings > CI settings`查找。

寻找更多完整的`.gitlab-ci.yml`语法，访问[.gitlab-ci.yml文档](http://git.daojia-inc.com/help/ci/yaml/README.md)

### Push `.gitlab-ci.yml` 文件

在创建完`.gitlab-ci.yml`文件，可以添加到git仓库、`push`到GitLab。

```
git add .gitlab-ci.yml
git commit -m "Add .gitlab-ci.yml"
git push origin master

```

现在如果去**Builds**页面，你会发现构建是`pending`状态。

同样可以去到**Commits**页面，会发现一个时钟的标志在提交记录旁边。

![提交记录](https://raw.githubusercontent.com/sunpeijun/gitlab-ci-doc/master/img/new_commit.png)

点击时钟图标，你将被引导到构建页面。

![提交状态](https://raw.githubusercontent.com/sunpeijun/gitlab-ci-doc/master/img/single_commit_status_pending.png)

注意那里有两个`job`在`pending`状态。红色的三角形表明还没有配置`Runner`构建。下一步选择`pending`状态的`job`配置`Runner`。

### 配置`Runner`

在GitLab中，`Runner`会去运行在`.gitlab-ci.yml`定义的构建项目。`Runner`可以是虚拟机，VPS，docker容器甚至是集群容器。GitLab通过API与`Runner`进行通信，所以只需要求`Runner`机器配置需要连接互联网。

`Runner`可以服务特定某个项目或服务多个GitLab项目。服务所有项目的叫`Shared Runner`

寻找更多关于`Runners`信息参考[Runners](http://git.daojia-inc.com/help/ci/runners/README.md)文档。

你可以在`Settings > Runners`找到关于你项目的`Runner`。建立一个`Runenr`很简单。官方GitLab支持的`Runner`采用`Go`来写，可以在[https://gitlab.com/gitlab-org/gitlab-ci-multi-runner](https://gitlab.com/gitlab-org/gitlab-ci-multi-runner)里面找到。

为了一个有功能的`Runner`你需要执行两步：
1. [安装](https://gitlab.com/gitlab-org/gitlab-ci-multi-runner/tree/master#installation)
2. [配置](http://git.daojia-inc.com/help/ci/runners/README.md#registering-a-specific-runner)

按照上面的链接设置自己的`Runner`或者使用下一节描述的`Shared Runner`

对于其他类型的语言编写的非官方`Runner`，参考[各种版本的GitLab Runner](https://about.gitlab.com/gitlab-ci/#gitlab-runner)

一旦`Runner`建立，你可以在项目的`Settings > Runners`页面看到。

![runners_activated](https://raw.githubusercontent.com/sunpeijun/gitlab-ci-doc/master/img/runners_activated.png)

### `Shared Runners`

如果你使用[GitLab.com](https://gitlab.com/),你可以使用GitLab提供的`Shared Runners`。

这些跑在GitLab基础设施上的特殊虚拟机可以编译任何项目。

启用`Shared Runners`需要去`Settings > Runners`点击`Enable shared runners`。

[点击阅读更多关于`Shared Runners`](http://git.daojia-inc.com/help/ci/runners/README.md)

### 查看构建状态

在配置`Runners`成功以后，你可以看到最后一次提交状态从`pending`到`running`到`success`或`failed`。

![builds_status](https://raw.githubusercontent.com/sunpeijun/gitlab-ci-doc/master/img/builds_status.png)

通过点击一个编译ID，你可以看到编译的日志。这是重要的诊断构建失败或预期行为不同的原因。

![build_log](https://raw.githubusercontent.com/sunpeijun/gitlab-ci-doc/master/img/build_log.png)

你可以在GitLab的各个页面查看任何提交的状态，`Commits`和`Merge Requests`

### 开启`build emails`

如果你想接收编译状态的电子邮件，应该在项目下启用`Builds Emails`服务设置。

更多信息阅读[编译emails服务文档](http://git.daojia-inc.com/help/project_services/builds_emails.md)

### 编译徽章

你可以使用以下链接访问构建构建徽章。

    http://example.gitlab.com/namespace/project/badges/branch/build.svg

很棒，你已经开始使用 GitLab CI。

### 实例

访问[examples README](http://git.daojia-inc.com/help/ci/examples/README.md)查看各种语言使用GitLab CI的例子。
