# Jenkins

## 1. 概述

Jenkins 是一个开源的持续集成和持续交付（CI/CD）工具，用于自动化软件开发的构建、测试和部署过程。它支持丰富的插件生态系统，可以与各种开发工具和技术集成。

## 2. 核心概念

### 2.1 架构组件
```mermaid
graph TB
    A[Jenkins Master] --> B[构建队列]
    A --> C[插件系统]
    A --> D[配置管理]
    A --> E[用户界面]
    
    F[Jenkins Agent] --> G[执行环境]
    F --> H[工作空间]
    F --> I[工具配置]
    
    A --> F
    
    style A fill:#e1f5fe
    style F fill:#f3e5f5
```

### 2.2 关键特性
- **可扩展性**: 丰富的插件生态系统（1800+ 插件）
- **分布式构建**: 支持主从架构，分布式执行任务
- **流水线即代码**: 使用 Groovy DSL 定义复杂流水线
- **社区支持**: 活跃的开源社区和商业支持
- **集成能力**: 与各种开发工具和云平台集成

## 3. 安装与配置

### 3.1 Docker 安装
```yaml
# docker-compose.yml
version: '3.8'

services:
  jenkins:
    image: jenkins/jenkins:lts-jdk17
    container_name: jenkins
    restart: unless-stopped
    privileged: true
    user: root
    ports:
      - "8080:8080"
      - "50000:50000"
    environment:
      - JAVA_OPTS=-Djenkins.install.runSetupWizard=false
      - JENKINS_ADMIN_ID=admin
      - JENKINS_ADMIN_PASSWORD=password
    volumes:
      - ./jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
      - /usr/bin/docker:/usr/bin/docker
    networks:
      - jenkins-network

networks:
  jenkins-network:
    driver: bridge
```

### 3.2 系统包安装 (Ubuntu)
```bash
#!/bin/bash
# install-jenkins-ubuntu.sh

# 安装 Java
sudo apt update
sudo apt install -y openjdk-17-jdk

# 添加 Jenkins 仓库
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

# 安装 Jenkins
sudo apt-get update
sudo apt-get install -y jenkins

# 启动 Jenkins
sudo systemctl enable jenkins
sudo systemctl start jenkins

# 查看初始密码
echo "初始管理员密码:"
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

### 3.3 初始配置
```bash
#!/bin/bash
# jenkins-setup.sh

# 等待 Jenkins 启动
while ! curl -s http://localhost:8080 > /dev/null; do
    sleep 5
done

# 获取初始密码
JENKINS_PASSWORD=$(sudo cat /var/lib/jenkins/secrets/initialAdminPassword)

# 自动安装推荐插件
curl -X POST -u "admin:${JENKINS_PASSWORD}" \
  -d '<install plugin="workflow-aggregator@latest" />' \
  -H 'Content-Type: text/xml' \
  http://localhost:8080/pluginManager/installNecessaryPlugins

# 创建管理员用户
curl -X POST -u "admin:${JENKINS_PASSWORD}" \
  -d 'script=
    import jenkins.model.*
    import hudson.security.*
    def instance = Jenkins.getInstance()
    def hudsonRealm = new HudsonPrivateSecurityRealm(false)
    hudsonRealm.createAccount("admin", "admin123")
    instance.setSecurityRealm(hudsonRealm)
    instance.save()
  ' \
  http://localhost:8080/scriptText
```

## 4. 系统配置

### 4.1 全局工具配置
```groovy
// 通过 Groovy 脚本配置全局工具
import jenkins.model.*
import hudson.tools.*

// 配置 JDK
def jdkInstaller = new hudson.tools.JDKInstaller("17", true)
def jdkInstallation = new hudson.model.JDK("jdk17", "", [jdkInstaller])
Jenkins.instance.getDescriptorByType(hudson.model.JDK.DescriptorImpl.class).setInstallations(jdkInstallation)

// 配置 Maven
def mavenInstaller = new hudson.tools.MavenInstaller("3.9.4")
def mavenInstallation = new hudson.model.Maven.MavenInstallation("maven3", "", [mavenInstaller])
Jenkins.instance.getDescriptorByType(hudson.model.Maven.DescriptorImpl.class).setInstallations(mavenInstallation)

// 配置 Git
def gitInstaller = new hudson.plugins.git.GitInstaller("latest")
def gitInstallation = new hudson.plugins.git.GitTool("git", "", [gitInstaller])
Jenkins.instance.getDescriptorByType(hudson.plugins.git.GitTool.DescriptorImpl.class).setInstallations(gitInstallation)

// 保存配置
Jenkins.instance.save()
```

### 4.2 系统设置
```groovy
// 系统配置脚本
import jenkins.model.*
import hudson.model.*

// 设置系统消息
Jenkins.instance.systemMessage = """
# Jenkins CI/CD 平台
欢迎使用 Jenkins 持续集成平台
"""

// 配置执行器数量
Jenkins.instance.numExecutors = 4

// 设置 Jenkins URL
Jenkins.instance.setRootUrl("http://jenkins.example.com:8080")

// 配置全局环境变量
def globalNodeProperties = Jenkins.instance.getGlobalNodeProperties()
def envVarsNodePropertyList = globalNodeProperties.getAll(hudson.slaves.EnvironmentVariablesNodeProperty.class)

def envVars = null
if (envVarsNodePropertyList.empty) {
    def newEnvVarsNodeProperty = new hudson.slaves.EnvironmentVariablesNodeProperty()
    globalNodeProperties.add(newEnvVarsNodeProperty)
    envVars = newEnvVarsNodeProperty.getEnvVars()
} else {
    envVars = envVarsNodePropertyList[0].getEnvVars()
}

envVars.put("DOCKER_HOST", "unix:///var/run/docker.sock")
envVars.put("MAVEN_OPTS", "-Xmx1024m -XX:MaxPermSize=512m")

// 保存配置
Jenkins.instance.save()
```

## 5. 节点管理

### 5.1 静态节点配置
```bash
#!/bin/bash
# setup-jenkins-agent.sh

# 安装 Java
sudo apt update
sudo apt install -y openjdk-17-jdk

# 创建 Jenkins 用户
sudo useradd -m -d /home/jenkins-agent -s /bin/bash jenkins-agent
sudo mkdir -p /home/jenkins-agent
sudo chown jenkins-agent:jenkins-agent /home/jenkins-agent

# 下载 agent.jar
wget http://jenkins-master:8080/jnlpJars/agent.jar -O /home/jenkins-agent/agent.jar

# 创建 systemd 服务
sudo tee /etc/systemd/system/jenkins-agent.service << 'EOF'
[Unit]
Description=Jenkins Agent
After=network.target

[Service]
User=jenkins-agent
WorkingDirectory=/home/jenkins-agent
ExecStart=/usr/bin/java -jar agent.jar -jnlpUrl http://jenkins-master:8080/computer/agent-name/slave-agent.jnlp -secret agent-secret-key
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

# 启动服务
sudo systemctl daemon-reload
sudo systemctl enable jenkins-agent
sudo systemctl start jenkins-agent
```

### 5.2 动态节点（Docker）
```groovy
// 配置 Docker Cloud
import com.nirima.jenkins.plugins.docker.*
import com.nirima.jenkins.plugins.docker.strategy.*
import io.jenkins.docker.*

def dockerCloud = new DockerCloud(
    "docker-cloud",
    [new DockerTemplate(
        new DockerTemplateBase("jenkins/agent:jdk17"),
        new DockerComputerLauncher(),
        new DockerOnceRetentionStrategy(10),
        "docker-agent-",
        "/home/jenkins",
        "10"
    )],
    "unix:///var/run/docker.sock"
)

Jenkins.instance.clouds.add(dockerCloud)
Jenkins.instance.save()
```

## 6. 流水线配置

### 6.1 声明式流水线
```groovy
// Jenkinsfile (Declarative Pipeline)
pipeline {
    agent {
        docker {
            image 'maven:3.9.4-eclipse-temurin-17'
            args '-v $HOME/.m2:/root/.m2'
        }
    }
    
    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
    }
    
    environment {
        NODE_VERSION = '18'
        DOCKER_REGISTRY = 'registry.example.com'
        CREDENTIALS_ID = 'docker-credentials'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh 'git log -1 --oneline'
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean compile -DskipTests'
            }
        }
        
        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'mvn test -Dtest=**/*Test'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        sh 'mvn verify -Dtest=**/*IT'
                    }
                }
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                    junit '**/target/failsafe-reports/*.xml'
                }
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    def version = sh(script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout', returnStdout: true).trim()
                    docker.build("${DOCKER_REGISTRY}/app:${version}")
                }
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'main'
            }
            steps {
                script {
                    def version = sh(script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout', returnStdout: true).trim()
                    withCredentials([usernamePassword(credentialsId: CREDENTIALS_ID, usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh """
                            docker login -u $USERNAME -p $PASSWORD $DOCKER_REGISTRY
                            docker push ${DOCKER_REGISTRY}/app:${version}
                        """
                    }
                    sh "kubectl set image deployment/app app=${DOCKER_REGISTRY}/app:${version} -n staging"
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
            script {
                def duration = currentBuild.durationString.replace(' and counting', '')
                echo "构建完成，耗时: ${duration}"
            }
        }
        success {
            emailext (
                subject: "构建成功: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                构建信息:
                项目: ${env.JOB_NAME}
                构建号: ${env.BUILD_NUMBER}
                状态: 成功
                耗时: ${currentBuild.durationString}
                详情: ${env.BUILD_URL}
                """,
                to: 'dev-team@example.com'
            )
        }
        failure {
            emailext (
                subject: "构建失败: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                构建信息:
                项目: ${env.JOB_NAME}
                构建号: ${env.BUILD_NUMBER}
                状态: 失败
                详情: ${env.BUILD_URL}
                """,
                to: 'dev-team@example.com'
            )
        }
    }
}
```

### 6.2 脚本式流水线
```groovy
// Jenkinsfile (Scripted Pipeline)
node('docker') {
    try {
        stage('Checkout') {
            checkout scm
            sh 'git log -1 --oneline'
        }
        
        stage('Build') {
            docker.image('maven:3.9.4-eclipse-temurin-17').inside('-v $HOME/.m2:/root/.m2') {
                sh 'mvn clean compile -DskipTests'
            }
        }
        
        stage('Test') {
            parallel(
                unit: {
                    docker.image('maven:3.9.4-eclipse-temurin-17').inside('-v $HOME/.m2:/root/.m2') {
                        sh 'mvn test -Dtest=**/*Test'
                        junit '**/target/surefire-reports/*.xml'
                    }
                },
                integration: {
                    docker.image('maven:3.9.4-eclipse-temurin-17').inside('-v $HOME/.m2:/root/.m2') {
                        sh 'mvn verify -Dtest=**/*IT'
                        junit '**/target/failsafe-reports/*.xml'
                    }
                }
            )
        }
        
        stage('Quality Gate') {
            withSonarQubeEnv('sonarqube') {
                docker.image('maven:3.9.4-eclipse-temurin-17').inside('-v $HOME/.m2:/root/.m2') {
                    sh 'mvn sonar:sonar'
                }
            }
            
            timeout(time: 1, unit: 'HOURS') {
                def qualityGate = waitForQualityGate()
                if (qualityGate.status != 'OK') {
                    error "质量门禁未通过: ${qualityGate.status}"
                }
            }
        }
        
        stage('Deploy') {
            if (env.BRANCH_NAME == 'main') {
                docker.withRegistry('https://registry.example.com', 'docker-credentials') {
                    def version = sh(script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout', returnStdout: true).trim()
                    def image = docker.build("app:${version}")
                    image.push()
                    image.push('latest')
                }
                
                withKubeConfig([credentialsId: 'k8s-credentials', serverUrl: 'https://k8s.example.com']) {
                    sh "kubectl set image deployment/app app=registry.example.com/app:${version} -n production"
                    sh "kubectl rollout status deployment/app -n production --timeout=300s"
                }
            }
        }
        
    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    } finally {
        stage('Cleanup') {
            cleanWs()
            echo "构建 ${currentBuild.result ?: 'SUCCESS'}"
        }
    }
}
```

## 7. 插件管理

### 7.1 常用插件列表
```groovy
// plugins.txt - 用于 Docker 安装
antisamy-markup-formatter:latest
authorize-project:latest
build-timeout:latest
cloudbees-folder:latest
credentials-binding:latest
email-ext:latest
git:latest
github-branch-source:latest
gradle:latest
ldap:latest
mailer:latest
matrix-auth:latest
pam-auth:latest
pipeline-github-lib:latest
pipeline-stage-view:latest
ssh-slaves:latest
timestamper:latest
workflow-aggregator:latest
ws-cleanup:latest
docker-workflow:latest
kubernetes:latest
sonarqube-scanner:latest
```

### 7.2 插件安装脚本
```bash
#!/bin/bash
# install-jenkins-plugins.sh

JENKINS_URL="http://localhost:8080"
JENKINS_USER="admin"
JENKINS_PASSWORD="admin123"

# 插件列表
PLUGINS=(
    "git"
    "docker-workflow"
    "kubernetes"
    "pipeline"
    "blueocean"
    "sonarqube-scanner"
    "email-ext"
    "ws-cleanup"
    "build-timeout"
    "credentials-binding"
)

# 安装插件
for PLUGIN in "${PLUGINS[@]}"; do
    echo "安装插件: $PLUGIN"
    curl -X POST -u "${JENKINS_USER}:${JENKINS_PASSWORD}" \
        "${JENKINS_URL}/pluginManager/install?plugin=${PLUGIN}&submit="
    sleep 5
done

# 重启 Jenkins
echo "重启 Jenkins..."
curl -X POST -u "${JENKINS_USER}:${JENKINS_PASSWORD}" \
    "${JENKINS_URL}/safeRestart"
```

## 8. 安全配置

### 8.1 安全加固
```groovy
// security.groovy
import jenkins.model.*
import hudson.security.*
import jenkins.security.s2m.AdminWhitelistRule

// 启用安全性
def instance = Jenkins.getInstance()

if (!instance.isUseSecurity()) {
    def strategy = new GlobalMatrixAuthorizationStrategy()
    
    // 管理员权限
    strategy.add(Jenkins.ADMINISTER, "admin")
    strategy.add(Jenkins.ADMINISTER, "authenticated")
    
    // 基本权限
    strategy.add(hudson.model.Hudson.READ, "authenticated")
    strategy.add(hudson.model.Item.READ, "authenticated")
    strategy.add(hudson.model.Item.DISCOVER, "authenticated")
    
    instance.setAuthorizationStrategy(strategy)
    
    // 启用安全域
    def realm = new HudsonPrivateSecurityRealm(false)
    realm.createAccount("admin", "admin123")
    instance.setSecurityRealm(realm)
    
    // 禁用脚本命令行
    instance.getDescriptor("jenkins.CLI").get().setEnabled(false)
    
    // 配置代理安全性
    instance.getInjector().getInstance(AdminWhitelistRule.class).setMasterKillSwitch(false)
    
    instance.save()
    println "安全性已启用"
}
```

### 8.2 凭据配置
```groovy
// credentials.groovy
import com.cloudbees.plugins.credentials.*
import com.cloudbees.plugins.credentials.common.*
import com.cloudbees.plugins.credentials.domains.*
import com.cloudbees.plugins.credentials.impl.*
import org.jenkinsci.plugins.plaincredentials.impl.*
import hudson.util.Secret

// 获取凭据域
def domain = Domain.global()

// 获取凭据存储
def store = Jenkins.instance.getExtensionList('com.cloudbees.plugins.credentials.SystemCredentialsProvider')[0].getStore()

// 添加 Docker Hub 凭据
def dockerCredentials = new UsernamePasswordCredentialsImpl(
    CredentialsScope.GLOBAL,
    "docker-hub-credentials",
    "Docker Hub Credentials",
    "dockeruser",
    "dockerpassword"
)
store.addCredentials(domain, dockerCredentials)

// 添加 GitHub Token
def githubToken = new StringCredentialsImpl(
    CredentialsScope.GLOBAL,
    "github-token",
    "GitHub API Token",
    Secret.fromString("ghp_yourtokenhere")
)
store.addCredentials(domain, githubToken)

// 添加 SSH 密钥
def sshKey = new BasicSSHUserPrivateKey(
    CredentialsScope.GLOBAL,
    "ssh-deploy-key",
    "deploy-user",
    new BasicSSHUserPrivateKey.DirectEntryPrivateKeySource("-----BEGIN PRIVATE KEY-----..."),
    "deploy-passphrase",
    "Deploy SSH Key"
)
store.addCredentials(domain, sshKey)

println "凭据配置完成"
```

## 9. 备份与恢复

### 9.1 自动备份脚本
```bash
#!/bin/bash
# jenkins-backup.sh

set -e

# 配置变量
BACKUP_DIR="/backup/jenkins"
RETENTION_DAYS=30
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="${BACKUP_DIR}/jenkins-backup-${TIMESTAMP}.tar.gz"

# 创建备份目录
mkdir -p "${BACKUP_DIR}"

# 停止 Jenkins（可选）
# sudo systemctl stop jenkins

# 执行备份
tar -czf "${BACKUP_FILE}" \
    --exclude="workspace" \
    --exclude="logs" \
    --exclude=".cache" \
    /var/lib/jenkins

# 备份插件列表
ls /var/lib/jenkins/plugins/*.jpi | awk -F/ '{print $NF}' | sed 's/\.jpi$//' > "${BACKUP_DIR}/plugins.txt"

# 启动 Jenkins（如果停止了）
# sudo systemctl start jenkins

# 清理旧备份
find "${BACKUP_DIR}" -name "jenkins-backup-*.tar.gz" -mtime +${RETENTION_DAYS} -delete

echo "备份完成: ${BACKUP_FILE}"

# 上传到云存储
aws s3 cp "${BACKUP_FILE}" "s3://jenkins-backups/"
aws s3 cp "${BACKUP_DIR}/plugins.txt" "s3://jenkins-backups/"
```

### 9.2 恢复脚本
```bash
#!/bin/bash
# jenkins-restore.sh

set -e

# 配置变量
BACKUP_FILE="/backup/jenkins/jenkins-backup-20230101_120000.tar.gz"

# 停止 Jenkins
sudo systemctl stop jenkins

# 清理旧数据
sudo rm -rf /var/lib/jenkins/*

# 执行恢复
sudo tar -xzf "${BACKUP_FILE}" -C /

# 设置权限
sudo chown -R jenkins:jenkins /var/lib/jenkins

# 启动 Jenkins
sudo systemctl start jenkins

echo "恢复完成"

# 重新安装插件
while ! curl -s http://localhost:8080 > /dev/null; do
    sleep 5
done

PLUGINS=$(cat /backup/jenkins/plugins.txt)
for PLUGIN in $PLUGINS; do
    echo "安装插件: $PLUGIN"
    curl -X POST -u "admin:admin123" \
        "http://localhost:8080/pluginManager/install?plugin=${PLUGIN}&submit="
    sleep 2
done

# 重启 Jenkins
curl -X POST -u "admin:admin123" "http://localhost:8080/safeRestart"
```
