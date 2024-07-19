# 激活指定 settings.xml 文件

这个常用于区分不同 maven 配置，假如你上班的时候偷偷赚外快接外包项目。那么外包项目和你公司使用的 maven 配置肯定不能是同一个，比如需要使用 `mvn deploy` 将 jar 上传到 maven 仓库咋办？到底是上传到公司呢还是外包呢？上传就上传吧，万一被发现了咋办？

这个时候就需要使用不同的 settings.xml 了，只需要为公司和外快项目分别设置一个 settings.xml，如：

```bash
$ ls ~/.m2
settings-outsourcing.xml settings-office.xml
```

之后当你需要打包上传时只需要指定不同的 settings 即可：

```bash
mvn deploy -s /HomePath/.m2/settings.xml
```

# 激活指定 profile

除了使用 `-s` 区分配置文件之外，还可以使用 `-P` 指定要激活的配置。

