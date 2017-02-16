
## runners

在GitLab CI中，Runner运行yaml文件。`runner`是一个独立的(虚拟的)机器，参与构建通过GitLab CI协调器API。

`runner`可以指定某个项目或服务GitLab中任何项目。用于所有项目的被称为`shared runner`。

理想情况下，GitLab Runner可以不和GitLab安装在同一机器上。阅读[需求文档](http://git.daojia-inc.com/help/install/requirements.md#gitlab-runner)获取更多信息。

### `Shared vs Specific Runners`
