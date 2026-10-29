<!--
 Copyright 1999-2026 Alibaba Group Holding Ltd.
 Licensed under the Apache License, Version 2.0 (the "License");
 you may not use this file except in compliance with the License.
 You may obtain a copy at http://www.apache.org/licenses/LICENSE-2.0
 Unless required by applicable law or agreed to in writing, software distributed
 under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR
 CONDITIONS OF ANY KIND, either express or implied. See the License for the
 specific language governing permissions and limitations under the License.
-->

# PigMesh Nacos 3.3.0

基于上游 Nacos `3.3.0` 标签（`e61ba821e025d713fc02d2020a338d4ce4251912`），
参考 `pig-mesh/nacos324` 的 `d542dcac8213cd19b1d6da1a028a47290d19f5aa` 迁移。
Java 包名和 Nacos 协议保持上游定义，Maven 坐标为 `io.github.pig-mesh.nacos:*:3.3.0`。

固定版本：Spring Boot 4.1.2、JRaft 1.4.0、Hessian 4.0.63。
其他依赖保留 Nacos 3.3.0 基线；外部 `com.alibaba.nacos:logback-adapter` 保持原坐标。

## 本地构建

使用 JDK 17 和 Maven 3.9：

```bash
mvn -B clean install -Dmaven.test.skip=true -Drat.skip=true
mvn -B -pl plugin/datasource,plugin-default-impl/nacos-default-datasource-plugin/nacos-datasource-plugin-dameng spotless:apply spotless:check
mvn -B -pl plugin-default-impl/nacos-default-datasource-plugin/nacos-datasource-plugin-dameng test -Drat.skip=true
```

`-Drat.skip=true` 用于绕过上游 `example/README.md` 已存在的许可证头缺失。
该参数不跳过编译、Checkstyle 或显式执行的测试。

默认 Reactor 构建 53 个库模块。可启动包使用：

```bash
mvn -B -Pdistribution,release-nacos -pl bootstrap -am package -Dmaven.test.skip=true -Drat.skip=true
```

输出为 `bootstrap/target/nacos-server.jar`，运行时插件位于 `distribution/plugins/`。
集成测试模块通过 `tests` profile 启用，例如：

```bash
mvn -Ptests,integration-test -am -pl test/openapi-test -DskipTests=false verify
```

## 达梦

插件坐标：`io.github.pig-mesh.nacos:nacos-datasource-plugin-dameng:3.3.0`。
使用 `nacos.plugin.datasource-dialect.type=dm`，默认驱动类为 `dm.jdbc.driver.DmDriver`。
部署方需自行提供 `com.dameng:DmJdbcDriver18:8.1.3.140`；该依赖为 provided、optional，
不会自动进入插件分发目录。

新数据库使用插件内 `META-INF/dm-schema.sql`（构建时同步到 `distribution/conf`）。
旧库升级要求见 [默认数据源方言规范](specs/zh-cn/plugin/default-datasource-dialect-plugin-spec.md)。
单元测试覆盖 SQL、参数顺序、分页、生成主键、默认驱动和 SPI；真实达梦实例执行仍需在目标环境验收。

## Maven Central 发布

`release` profile 生成扁平化 POM、源码、Javadoc 和 GPG 签名；排除五个聚合 POM，
并跳过 Central 已存在组件。凭据和签名配置只放在本机 Maven/GPG 安全配置中。

正式发布前先执行仅本地的发布制品检查：

```bash
mvn -B install -Prelease -Dmaven.test.skip=true -Drat.skip=true -Dgpg.skip=true -Dcentral.skipPublishing=true
```

确认版本尚未发布且验收完成后，才执行 `mvn clean deploy -Prelease -Dmaven.test.skip=true -Drat.skip=true`。
部署命令结束后还须核对 Central 最终状态、公共仓库制品和空白本地仓库消费结果。
本次开发验证不代表已发布到 Maven Central。
