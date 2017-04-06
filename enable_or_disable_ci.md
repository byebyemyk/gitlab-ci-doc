
## 启用或禁用GitLab CI

要有效的使用GitLab CI，你需要一个合法的[.gitlab-ci.yml](http://git.xxx-inc.com/help/ci/yaml/README.md)文件和一个正确设置的[runner](http://git.xxx-inc.com/help/ci/runners/README.md)。你可以通过阅读[开始使用 GitLab CI](quick_start.md)开始。如果你使用其他的CI服务器像`Jenkins`或者`Drone CI`，我们建议禁用GitLab CI防止提交状态出现出现冲突。

在GitLab 8.2，GitLab主要在项目`/builds`页面显示。禁用GitLab不会删除任何先前的构建。尽管 Gitlab CI从左边菜单栏隐藏，但实际上依然可以从 `/builds`页面访问到。

GitLab CI在新安装的时候是默认启用的，可以在每个项目中被禁用，或者在站点范围内通过修改设置`gitlab.yml`和`gitla.rb`

### 每个项目的用户设置

开启或禁用GitLab CI可以在`Features`下的`Builds`区域设置，还有`Issues`、`Merge Requests`、`Wiki`、`Snippets`。选择或取消选择复选框，点击`Save`生效。

![features_settings](https://raw.githubusercontent.com/sunpeijun/gitlab-ci-doc/master/img/features_settings.png)

### 站点管理员设置

通过修改`gitlab.yml`和`gitlab.rb`设置，你可以禁用全站的GitLab CI。

注意事项：
1. 禁用GitLab CI将会影响新创建的项目，在修改之前开启的项目会像原来一样工作。
2. 即使你禁用GitLab CI，用户仍然能够在项目设置中开启。

从源码安装的可以用编辑器打开`gitlab.yml`，设置`build`为`false`

```
## Default project features settings
default_projects_features:
  issues: true
  merge_requests: true
  wiki: true
  snippets: false
  builds: false
```

保存文件并且重启Gitlab：` sudo service gitlab restart `

对于`Omnibus installations`安装，编辑`/etc/gitlab/gitlab.rb`并且添加一行：

    gitlab_rails['gitlab_default_projects_features_builds'] = false

保存文件并重新配置`GitLab`: `sudo gitlab-ctl reconfigure`
