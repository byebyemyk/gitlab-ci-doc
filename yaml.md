
## `.gitlab-ci.yml`配置

本文档描述`.gitlab-ci.yml使用`，GitLab `Runner`使用这个文件去管理项目的构建。

如果你需要GitLab CI的快速介绍，点击[快速指南](http://git.daojia-inc.com/help/ci/quick_start/README.md)

**表格内容**通过[DocToc](https://github.com/thlorenz/doctoc)生成

* `.gitlab-ci.yml`
    * `image and services`
    * `before_script`
    * `after_script`
    * `stages`
    * `types`
    * `variables`
    * `cache`
        * `cache:key`

* `Jobs`
    * `script`
    * `stage`
    * `job variables`
    * `only and except`
    * `tags`
    * `when`
    * `artifacts`
        * `artifacts:name`
        * `dependencies`
        * `before_script and after_script`

* `Hidden jobs`
* `Special YAML features`
    * `Anchors`

* `Validate the .gitlab-ci.yml`
* `Skipping builds`
* `Examples`

### `.gitlab-ci.yml`

从7.12版本开始，GitLab CI使用[YAML](https://en.wikipedia.org/wiki/YAML)文件(`.gitlab-ci.yml`)作为项目的配置。放置在项目的根目录并包含定义如何构建项目。

YAML文件定义一些包含约束和运行时间的`job`，`job`定义为包含名字的顶级元素，并且总是包含`script`子句。

```
job1:
  script: "execute-script-for-job1"

job2:
  script: "execute-script-for-job2"
```

上面的例子是两个独立`job`最简单的CI配置，每个`job`执行不同的命令。

当然命令可以直接运行（`./configure; make; make install`），或者运行仓库中的脚本（`test.sh`）

`Job`用来创建构建，然后被`Runner`采用和在`Runner`环境中执行。重要的是，每个工作都是相互独立运行。

YAML的语法可以使用更复杂的`job`说明。

```
image: ruby:2.1
services:
  - postgres

before_script:
  - bundle install

after_script:
  - rm secrets

stages:
  - build
  - test
  - deploy

job1:
  stage: build
  script:
    - execute-script-for-job1
  only:
    - master
  tags:
    - docker
```

注意：有一些保留的**关键字**不能用作`job`名称：

关键字 | 必需 | 描述
---|--- |---
`image` | no | 使用docker镜像，详情见[Use Docker]
`services` | no | 使用docker服务，详情见[Use Docker]
`stages` | no | 定义构建的阶段
`types` | no | `stages`的别名
`before_script` | no | 定义运行在每个`job`脚本前的命令
`after_script` | no | 定义运行在每个`job`脚本后的命令
`variables` | no | 定义构建变量
`cache` | no | 定义在后续运行中可以被缓存的文件

### `image and services`

允许指定自定义的`Docker`镜像和一系列的服务列表。这些功能的配置在一个 [单独的文档](http://git.daojia-inc.com/help/ci/docker/README.md)

### `before_script`

`before_script`用来定义一些命令，在所有构建之前运行，包括`deploy`。可以是一个数组或一个多行的字符串。

### `after_scripts`

> 注意：在GitLab 8.7中引入，需要`GitLab Runner` v1.2（还没有发布）

`after_script`用来定义一些命令，在所有构建之后运行，可以是一个数组或一个多行的字符串。

### `stages`

`stages`用来定义`job`的编译阶段。阶段的规范允许灵活的多级管道。

阶段中的元素顺序定义构建的执行顺序。
1. 相同阶段的构建并行的执行。
2. 下一阶段的构建运行在前一阶段成功完成上。

让我们考虑下面的示例，它定义了3个阶段:

```
stages:
  - build
  - test
  - deploy
```

1. 首先所有的`build`阶段的`job`是并行执行的。
2. 如果`build`阶段成功，`test`阶段并行执行
3. 如果`test`阶段成功，`deploy`阶段并行执行
4. 如果`deploy`阶段成功，`commit`标记为`success`
5. 如果之前的`job`失败，`commit`标记为`failed`并且接下来的阶段不会执行

值得一提的两个边界情况：
1. 如果没有在`.gitlab-ci.yml`中定义`stages`，默认的`build`,`test`和`deploy`将会被默认使用作为`job's stage`
2. 如果一个`job`没有指定`stage`，将会指定`test`阶段。

### `types`

`stages`的别名

### `variables`

> 注意：GitLab Runner v0.5.0引入

GitLab CI可以在`.gitlab-ci.yml`文件中添加变量设置在构建环境中。变量存储在git仓库，用于存储非敏感项目配置，例如：

```
variables:
  DATABASE_URL: "postgres://postgres@postgres/my_database"
```

这些变量可以随后用于执行的所有命令和脚本。

`YAML-defined`变量同样设置已经创建的`service containers`,可以允许微调。

变量可以同样用来定义`job level`

### `cache`

> 注意：GitLab Runner v0.7.0引入

`cache`用来在构建间缓存指定的文件或目录。

**默认情况下缓存在每份`job`每个分支都开启**

如果缓存在`job`之外定义，会被标记为全局的并且所有的`job`都会使用它的定义。

缓存所有`binaries`和`.config`文件：

```
rspec:
  script: test
  cache:
    paths:
    - binaries/
    - .config
```

缓存所有`Git untracked`文件：

```
rspec:
  script: test
  cache:
    untracked: true
```

缓存所有`binaries`目录下Git untracked`文件：

```
rspec:
  script: test
  cache:
    untracked: true
    paths:
    - binaries/
```

局部定义的缓存会覆盖全局定义的缓存。这将只会缓存`binaries/`目录：

```
cache:
  paths:
  - my/files

rspec:
  script: test
  cache:
    paths:
    - binaries/
```

缓存尽力而为，不要期望永远存在。具体实现细节访问 `GitLab Runner`

### `cache:key`

> 注意：GitLab Runner v1.0.0引入

`key`指令定义缓存之间的关联。可以允许所有的`job`有单个缓存，缓存每个`job`，缓存每个分支，或者其他你认为适当的方式。

这允许你调整缓存，可以在不同的`job`或不同的分支之间缓存。

`cache:key`变量可以使用任何预设变量。

**配置示例**

开启每个`job`缓存：

```
cache:
  key: "$CI_BUILD_NAME"
  untracked: true
```

开启每个分支缓存：

```
cache:
  key: "$CI_BUILD_REF_NAME"
  untracked: true
```

开启每个`job`和每个分支缓存：

```
cache:
  key: "$CI_BUILD_NAME/$CI_BUILD_REF_NAME"
  untracked: true
```

开启每个分支和每个状态缓存：

```
cache:
  key: "$CI_BUILD_STAGE/$CI_BUILD_REF_NAME"
  untracked: true
```

如果你使用 `Windows Batch` 运行shell脚本，需要把`$`换成`%`:

```
cache:
  key: "%CI_BUILD_STAGE%/%CI_BUILD_REF_NAME%"
  untracked: true
```

### `Jobs`

`.gitlab-ci.yml`允许你定义不限数量的`job`.每个`job`必须有一个唯一的名字，不能是上面提到的关键词之一。`job`被定义构建构建行为的参数列表。

```
job_name:
  script:
    - rake spec
    - coverage
  stage: test
  only:
    - master
  except:
    - develop
  tags:
    - ruby
    - postgres
  allow_failure: true
```

关键字 | 必需 | 描述
---|--- |---
`script` | yes | 定义可以被`Runner`执行的shell script
`image` | no | 使用docker镜像，详情见[Use Docker]
`services` | no | 使用docker服务，详情见[Use Docker]
`stages` | no | 定义构建的阶段（默认test）
`types` | no | `stages`的别名
`variables` | no | 定义`job level`的变量
`only` | no | 定义编译时创建git引用列表
`except` | no | 定义编译不创建的git引用列表
`tags` | no | 定义列表标签用于选择`Runner`
`allow_failure` | no | 允许构建失败，构建失败不影响提交状态
`when` | no | 定义编译时机，可以`on_success`，`on_failure`，`always`
`dependencies` | no | 定义当前构建的依赖
`artifacts` | no | 定义构建列表
`cache` | no | 定义可以在后续运行中缓存的列表文件
`before_script` | no | 定义运行在每个`job`脚本前的命令
`after_script` | no | 定义运行在每个`job`脚本后的命令
`variables` | no | 定义构建变量

### `script`

`script`可以被`Runner`执行的shell脚本。例如：

```
job:
  script: "bundle exec rspec"
```

这个参数可以包含多个命令通过使用数组。

```
job:
  script:
    - uname -a
    - bundle exec rspec
```

### `stage`

`stage`允许分组编译不同的`stage`。构建相同的阶段是并行执行的。

### `only and except`

`only`和`except`两个参数可以用来设置`job`编译的时间。
1. `only`定义被`job`编译`branches`和`tags`的名字。
2. `except`定义不被被`job`编译`branches`和`tags`的名字。

这里有一些规则适用于参考使用：

* `only`和`except`可并存的。如果`only`和`except`都在`job specification`定义，资源会被`only`和`except`过滤。
* `only`和`except`允许使用正则表达式。
* `only`和`except`允许使用特殊的关键字：`branches`,`tags`和`triggers`
* `only`和`except`允许过滤指定的仓库地址

在下面的例子中，`job`仅仅会运行带`issue-`的资源，而所有的分支都会被忽略。

```
job:
  # use regexp
  only:
    - /^issue-.*$/
  # use special keyword
  except:
    - branches
```

在这个例子中，`job`仅会运行标记的资源，或是通过API触发器显式地请求。

```
job:
  # use special keywords
  only:
    - tags
    - triggers
```

~~仓库地址用来做`jobs`执行，而且父仓库没有forks。~~

```
job:
  only:
    - branches@gitlab-org/gitlab-ce
  except:
    - master@gitlab-org/gitlab-ce
```

上面的示例将会在`gitlab-org/gitlab-ce`分支运行`job`，`master`除外。

### `job variables`




[Use Docker]: http://git.daojia-inc.com/help/ci/docker/README.md  
