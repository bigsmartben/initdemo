## 模板A：java21-springboot-dul

### 1. 核心技术栈 (Core Stack)
- **Language**: Java 21 (Eclipse Temurin LTS)
- **Framework**: Spring Boot 3.4.x
- **Build Tool**: Maven 3.9.x + Maven Wrapper (mvnw)
- **Architecture**: Multi-Module Monolith (多模块单应用架构)
- **Default_Port**: 8080

### 2. 工程架构 (Architecture)
脚手架初始化后，项目目录结构必须符合以下规范：
```text
jzx-project-root/               # 根项目 (Parent)
├── .mvn/                       # Maven Wrapper 核心配置
├── mvnw                        # Linux/macOS 启动脚本
├── mvnw.cmd                    # Windows 启动脚本
├── pom.xml                     # 根 POM: 聚合模块与依赖锁定
├── jzx-common/                 # 模块1: 通用工具与常量
├── jzx-domain/                 # 模块2: 领域模型与 Repository 接口
├── jzx-application/            # 模块3: Service 业务逻辑实现
└── jzx-api/                    # 模块4: Controller 入口与配置
    ├── src/main/resources/     # application.yml
    ├── pom.xml
    └── Dockerfile              # 容器构建定义

```

### 3. 【核心代码实现】 (Implementation Source)

*注：AI 在执行 /init-jzx 指令时，若本地缺失对应文件，需从此章节提取内容并创建。*

#### A. 环境检测脚本 (env-check.sh)

用于扫描本地环境并输出 `env_report.json`：

```bash
#!/bin/bash
REPORT_FILE="env_report.json"
echo "{" > $REPORT_FILE
# JDK 检查
JAVA_VER=$(java -version 2>&1 | awk -F '\"' '/version/ {print $2}')
echo "  \"java_version\": \"$JAVA_VER\"," >> $REPORT_FILE
# Docker 检查
docker info > /dev/null 2>&1 && D_STAT="running" || D_STAT="not_found"
echo "  \"docker_status\": \"$D_STAT\"," >> $REPORT_FILE
# 端口 8080 占用检查
PORT_8080=$(lsof -i :8080 | grep LISTEN | awk '{print $1}')
[ -z "$PORT_8080" ] && P_STAT="available" || P_STAT="occupied"
echo "  \"port_8080_status\": \"$P_STAT\"," >> $REPORT_FILE
echo "  \"port_8080_owner\": \"$PORT_OWNER\"" >> $REPORT_FILE
echo "}" >> $REPORT_FILE

```

#### B. 容器化构建定义 (Dockerfile)

支持多阶段构建，确保镜像体积最小化：

```dockerfile
# Stage 1: 容器内编译
FROM eclipse-temurin:21-jdk-jammy AS builder
WORKDIR /source
# 1. 复制构建引擎
COPY .mvn/ .mvn
COPY mvnw pom.xml ./
# 2. 复制各模块定义
COPY jzx-common/pom.xml jzx-common/
COPY jzx-domain/pom.xml jzx-domain/
COPY jzx-application/pom.xml jzx-application/
COPY jzx-api/pom.xml jzx-api/
# 3. 预下载依赖
RUN ./mvnw dependency:go-offline -B
# 4. 编译全量代码
COPY . .
RUN ./mvnw clean package -DskipTests -pl jzx-api -am

# Stage 2: 运行时环境
FROM eclipse-temurin:21-jre-jammy
WORKDIR /app
COPY --from=builder /source/jzx-api/target/*.jar app.jar
ENV SERVER_PORT=8080
EXPOSE ${SERVER_PORT}
ENTRYPOINT ["java", "-Djava.security.egd=file:/dev/./urandom", "-jar", "app.jar"]

```

#### C. 本地编排配置 (docker-compose.yml)

用于一键启动应用环境：

```yaml
services:
  jzx-app:
    build:
      context: .
      dockerfile: jzx-api/Dockerfile
    ports:
      - "${LOCAL_PORT:-8080}:${SERVER_PORT:-8080}"
    environment:
      - SPRING_PROFILES_ACTIVE=prod

```

### 4. 核心配置文件参考

#### Maven 根项目 pom.xml 关键配置

```xml
<properties>
    <java.version>21</java.version>
    <spring-boot.version>3.4.1</spring-boot.version>
</properties>
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>${spring-boot.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

```

---

## 模板B：python37-flask (待完善)

*(此部分留空，待后续扩展具体规范)*

```

---
