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

除了使用 `-s` 区分配置文件之外，还可以使用 `-P` 指定要激活的配置。`-P` 参数同时作用于项目 pom.xml 文件和 maven 的 settings.xml 文件。用于激活指定 profiles，可以根据不同的构建环境和需求实现对项目的定制化配置。

比如在构建项目时需要区分开发环境和发布环境（如不同环境使用不同依赖或插件），就可以在 pom.xml 中使用 profiles 配置：

```xml
<project>
    ...
    <profiles>
        <profile>
            <id>dev</id>
            <properties>
                <env>development</env>
            </properties>
            <dependencies>
                <dependency>
                    <groupId>com.example</groupId>
                    <artifactId>dev-dependency</artifactId>
                    <version>1.0.0</version>
                </dependency>
            </dependencies>
        </profile>
        <profile>
            <id>prod</id>
            <properties>
                <env>production</env>
            </properties>
            <dependencies>
                <dependency>
                    <groupId>com.example</groupId>
                    <artifactId>prod-dependency</artifactId>
                    <version>1.0.0</version>
                </dependency>
            </dependencies>
        </profile>
    </profiles>
    ...
</project>
```

在使用时可以使用 `-P` 来指定要激活的环境：

```bash
# 开发环境
mvnd -P dev

# 产线环境
mvnd -P prod
```

`-P` 不仅仅作用 pom.xml，还同时作用在 settings.xml 文件中。当执行 `mvnd -P dev` 命令时，不仅仅会查找 pom.xml 中的 dev 配置，还会查找 settings.xml 的同名配置。作用 settings.xml 文件时，等价于 `<activeProfiles>` 元素。

下面是 settings.xml 示例：

```xml
<settings>
    <profiles>
        <profile>
            <id>dev</id>
            <!-- 配置内容 -->
        </profile>
        <profile>
            <id>prod</id>
            <!-- 配置内容 -->
        </profile>
    </profiles>
    <activeProfiles>
        <activeProfile>dev</activeProfile>
    </activeProfiles>
</settings>
```

默认使用的是 dev 配置，当需要发布时可以使用 `-P` 参数指定指定 prod 配置。当然了，可以同时激活多个配置（示例如下）。

- 同时激活 dev 和 prod：

```bash
mvn -P dev,prod
```

该命令会同时使用 pom.xml 中的 dev、prod 配置及 settings.xml 中的 dev 和 prod 配置。

- 禁用 dev，启用 prod：

```bash
mvn -P !dev,prod
```

该命令会禁用 dev 配置，并仅仅使用 prod 配置。

是不是很灵活？当然了还可以配合 `-s` 一起使用：

```bash
mvn -s /path/settings.xml -P profile_id <command>
```


