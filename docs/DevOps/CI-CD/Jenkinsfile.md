# Jenkinsfile

：一个文本文件，用于描述 Jenkins 的 Pipeline Job ，可以被 Groovy 解释器执行。
- 在 Jenkins 上编辑 Job 时，可以通过 Web 表单填写多种配置参数。
  - 使用 Jenkinsfile 时，可以将大部分 Web 表单的配置参数用代码描述，更灵活，容易迁移。
  - 每执行一次 Jenkinsfile ，Jenkins 会自动识别其中的配置参数，导入相应的 Web 表单中，实现向下兼容。
    - 不过，有的配置参数是 Jenkinsfile 独有的，不支持导入。
- Jenkinsfile 有两种写法：
  - 脚本式（Scripted Pipeline）：将流水线定义在 node{} 中，内容为可执行的 Groovy 脚本。
  - 声明式（Declarative Pipeline）：将流水线定义在 pipeline{} 中，内容为声明式的语句。本文采用这种写法。
- 所有 Pipeline Job 的 Web 页面中都有一个名为 "流水线语法" 的链接，点击之后可以查看一些关于 Pipeline 的帮助文档。
  - 比如可以使用 "片段生成器" ，将 Web 表单中的配置参数转换成流水线代码。

## 例

```groovy
pipeline {
    agent {                     // 声明使用的节点
        label 'master'
    }
    environment {               // 定义环境变量
        PORT = '80'
    }
    options {
        timestamps()
        timeout(time: 5, unit: 'MINUTES')
    }
    stages {
        stage('拉取代码') {      // 开始一个阶段
            environment {       // 定义该阶段的环境变量
                GIT_BRANCH = 'dev'
            }
            steps {             // 执行一些命令
                sh "git checkout $GIT_BRANCH"
                echo '已切换分支'
            }
        }
        stage('构建镜像') {
            steps {
                docker build -t ${image_hub}/${image_project}/${build_image_name}:${build_image_tag} .
                docker push ${image_hub}/${image_project}/${build_image_name}:${build_image_tag}
                docker image rm ${image_hub}/${image_project}/${build_image_name}:${build_image_tag}
            }
        }
        stage('测试') {
            steps {
                echo '测试中...'
                echo '测试完成'
            }
        }
    }
    post {
        always {            // 任务结束时总是执行以下操作
            deleteDir()     // 递归地删除当前目录。这里只会删除全局 agent 的 ${env.WORKSPACE} 目录，不会删除局部 agent 的
        }
    }
}
```
- 区分大小写。
- 用 // 声明单行注释。
- 每个 {} 的内容不能为空。

## 变量

- Groovy 支持在字符串中用 `$` 插入变量的值，用 `${}` 插入表达式的值。如下：
  ```groovy
  script {
      ID = "1"            // 创建变量
      NAME = "man" + ID   // 拼接字符串
      echo NAME           // 直接打印 Groovy 的变量
      echo "$ID"          // 字符串定界符为双引号时，支持用 $ 插入变量或表达式的值
      echo '$ID'          // 字符串定界符为双引号时，不支持用 $ 取值
      sh "echo $ID"       // 执行 sh 语句，字符串定界符为双引号时， $ 会先读取 Groovy 变量，再作为 Shell 命令执行
      sh 'echo $ID'       // 执行 sh 语句，字符串定界符为双引号时，会直接作为 Shell 命令执行，因此 $ 会读取 Shell 变量
  }
  ```
  - 如果只想在字符串中使用 `$` 字符，则需要转义为 `\$` 。

- Jenkins 在执行 Jenkinsfile 之前，会先渲染以双引号作为定界符的字符串，如果其中存在 $ 符号，则尝试对 Groovy 解释器中的变量进行取值。
  - 如果使用的 Groovy 变量不存在，则报出 Groovy 的语法错误 `groovy.lang.MissingPropertyException: No such property` 。
  - 以单引号作为定界符的字符串不会渲染，而是直接使用。

### 环境变量

- 在 environment{} 中可以定义环境变量，它们会保存为 Groovy 变量和 Shell 变量。如下：
  ```groovy
  stage('测试') {
      environment {
          ID = 1
      }
      steps {
          sh "echo $ID"
          sh 'echo $ID'
      }
  }
  ```
  - 定义在 pipeline.environment{} 中的环境变量会作用于全局，而定义在 stage.environment{} 中的只作用于该阶段。

- 在 environment{} 中可以导入 Jenkins 的凭据作为环境变量：
  ```groovy
  environment {
      ACCOUNT1 = credentials('account1')
  }
  ```
  假设该凭据是 `Username With Password` 类型，值为 `admin:123456` ，则 Jenkins 会在 Shell 中加入三个环境变量：
  ```sh
  ACCOUNT1=admin:123456
  ACCOUNT1_USR=admin
  ACCOUNT1_PSW=123456
  ```
  - 读取其它类型的凭据时，建议打印出 Shell 的所有环境变量，从而发现 Jenkins 加入的环境变量的名字。
  - 为了保密，如果直接将上述变量打印到 stdout 上，Jenkins 会将它们的值显示成 `****` 。

### 构建参数

- 可以给 Job 声明构建参数，它们会保存为 Groovy 变量和 Shell 变量。
  - 用户在启动 Job 时，必须传入构建参数，除非它们有默认值。
- Pipeline Job 可以在 pipeline.parameters{} 中以代码的形式定义构建参数，而其它类型的 Job 只能在 Jenkins Web 页面中定义构建参数。如下：
    ```groovy
    pipeline {
        agent any
        parameters {
            booleanParam(name: 'A', defaultValue: true, description: '')   // 布尔参数
            choice(name: 'E', choices: ['A', 'B', 'C'], description: '')   // 单选参数，输入时会显示成下拉框
            string(name: 'B', defaultValue: 'Hello', description: '')      // 字符串参数，在 Web 页面上输入时不能换行
            text(name: 'C', defaultValue: 'Hello\nWorld', description: '') // 文本参数，输入时可以换行
            password(name: 'D', defaultValue: '123456', description: '')   // 密文参数，输入时会显示成密文
            file(name: 'f1', description: '')                              // 文件参数，输入时会显示文件上传按钮
        }
        stages {
            stage('Test') {
                steps {
                    echo "$A"   // 也可通过 $params.A 的格式读取构建参数，避免与环境变量重名
                }
            }
        }
    }
    ```
  - 如果定义了 parameters{} ，则会移除在 Jenkins Web 页面中定义的、在上游 Job 中定义的构建参数。
  - 每次修改了 parameters{} 之后，要执行一次 Job 才会在 Jenkins Web 页面上生效。
  - password 类型的参数虽然在输入时显示成密文，但打印到终端上时会显示成明文，不如 Jenkins 的 credentials 安全。
  - file 类型的参数上传的文件会存储到 `${workspace}/${job_name}/f1` 路径处，而用 `$f1` 可获得上传的文件名。
    - pipeline job 的文件参数功能无效，不能上传文件。可采用以下两种替代方案：
      - 创建一个普通类型的 job ，供用户上传文件，保存到主机的 /tmp 目录下。然后让其它 job 从这里拷贝文件。
      - 在 Jenkins 之外搭建一个文件服务器，供用户上传文件。然后让其它 job 从这里下载文件。这样上传文件时麻烦些，但方便跨主机拷贝文件。

- 在 shell 命令中调用构建参数时，可能被注入攻击。
  - 比如脚本执行 `ls $file` ，而用户输入构建参数 `file=a;rm -rf *` 就会注入攻击。
  - 如果让用户输入 booleanParam、choice 类型的构建参数，则在 Web 页面上只能选择有限的值。
    - 即使用户通过 HTTP API 输入构建参数，Jenkins 也会自动检查参数的值是否合法。如果不合法，则采用该参数的默认值。
  - 如果让用户输入 string、text 类型的构建参数，则应该过滤之后再调用。如下：
    ```groovy
    parameters {
        string(name: 'file')
    }
    environment {
        file = file.replaceAll('[^0-9A-Za-z /._-]', '_')  // 替换构建参数中的特殊字符
    }
    ```

### 内置变量

- 可以通过 env 字典访问 Pipeline 内置的环境变量。如下：
  ```sh
  env.JENKINS_HOME    # Jenkins 部署的主目录
  env.NODE_NAME       # 节点名
  env.WORKSPACE       # 在当前节点上的工作目录

  env.JOB_NAME        # 任务名
  env.JOB_URL         # 任务链接
  env.BUILD_NUMBER    # 构建编号
  env.BUILD_URL       # 构建链接

  env.BRANCH_NAME     # 分支名
  env.CHANGE_AUTHOR   # 版本的提交者
  env.CHANGE_URL      # 版本的链接
  ```
  - 这些变量的值都是 String 类型。
  - 这些变量可以按以下格式读取：
    ```groovy
    script {
        echo env.NODE_NAME              // 在 Groovy 代码中，通过 env 字典读取
        echo "${env.NODE_NAME}"         // 在字符串中，通过 $ 取值

        sh "echo ${env.NODE_NAME}"
        sh 'echo $NODE_NAME'            // env 字典的内容会导入 Shell 的环境变量，因此可以在 shell 中直接读取
    }
    ```

- 可以通过 currentBuild 字典获取当前的构建信息。如下：
  ```sh
  echo currentBuild.displayName       # Build 的名称，格式为 #number
  echo currentBuild.fullDisplayName   # Build 的全名，格式为 JOB_NAME #number
  echo currentBuild.description       # Build 的描述，默认为 null
  echo currentBuild.duration          # Build 的持续时长，单位 ms
  echo currentBuild.result            # Build 的结果，如果构建尚未结束，则返回值为 null
  echo currentBuild.currentResult     # Build 的当前结果。开始执行时为 SUCCESS ，受每个 stage 影响，不会为 null
  ```
  - 只有 displayName、description 变量支持修改。修改其它变量时会报错：`RejectedAccessException: No such field`
  - 例：修改本次构建的名称
    ```groovy
    script {
        currentBuild.displayName = "#${env.BUILD_NUMBER} branch=${env.BRANCH}"
    }
    ```

- 可以通过 params 字典获取 Pipeline 的构建参数。如下：
  ```sh
  params.A
  params.B
  ```

## agent{}

：用于控制在哪个 Jenkins 代理上执行流水线。
- 可用范围：
  - 在 pipeline{} 中必须定义 agent{} ，作为所有 stage{} 的默认代理。
  - 在单个 stage{} 中可选定义 agent{} ，只作用于该阶段。
- agent 常见的几种定义格式：
  ```groovy
  agent none          // 不设置全局的 agent ，此时要在每个 stage{} 中单独定义 agent{}
  ```
  ```groovy
  agent any           // 让 Jenkins 选择任一代理
  ```
  ```groovy
  agent {
      label 'master'  // 选择指定名字的代理
  }
  ```
  ```groovy
  agent {
      node {          // 选择指定名字的代理，并指定工作目录
          label 'master'
          customWorkspace '/opt/jenkins_home/workspace/test1'
      }
  }
  ```
- 可以创建临时的 docker 容器作为 agent ：
  ```groovy
  agent {
      docker {
          // label 'master'
          image 'centos:7'
          // args  '-v /tmp:/tmp'
      }
  }
  ```
  - 这会在指定节点（默认是 master 节点）上创建一个 docker 容器，执行 pipeline ，然后自动删除该容器。
  - 该容器的启动命令的格式如下：
    ```sh
    docker run -t -d -u 1000:1000 \
        -w /opt/.jenkins/workspace/test_pipeline@2 \    # 因为宿主机上已存在该项目的工作目录了，所以加上后缀 @2 避免同名
        -v /opt/.jenkins/workspace/test_pipeline@2:/opt/.jenkins/workspace/test_pipeline@2:rw,z \
        -e ********                                     # 传入 Jenkins 的环境变量
        centos:7 cat
    ```

## stages{}

pipeline{} 流水线的主要内容写在 stages{} 中，其中可以定义一个或多个 stage{} ，表示执行的各个阶段。
- Jenkins 会按先后顺序执行各个 stage{} ，并在 Web 页面上显示执行进度。
- 每个 stage{} 的名称不能重复。
- 每个 stage{} 中有且必须定义一个以下类型的语句块：
  ```sh
  stages{}
  steps{}
  matrix{}
  parallel{}
  ```
- 例：
  ```groovy
  stages {
      stage('测试 1') {
          steps {...}
      }
      stage('测试 2') {
          stages('嵌套阶段') {
              stage('单元测试 1') {
                  steps {...}
              }
              stage('单元测试 2') {
                  steps {...}
              }
          }
      }
  }
  ```

## steps{}

在 steps{} 中可以使用多种 DSL 语句。
- [官方文档](https://www.jenkins.io/doc/pipeline/steps/)

### echo

：用于显示字符串。
- 例：
  ```groovy
  steps {
      echo 'Hello'
  }
  ```
- echo 语句只能显示 String 类型的值，使用 println 可以显示任意类型的值。
- 使用字符串时，要用双引号 " 或单引号 ' 包住（除非是纯数字组成的字符串），否则会被当作变量取值。
  - 例如：`echo ID` 会被当作 `echo "$ID"` 执行。
  - 使用三引号 """ 或 ''' 包住时，可以输入换行的字符串。

### sh

：用于执行 shell 命令。
- 例：
  ```groovy
  steps {
      sh "echo Hello"
      sh 'echo World'
      sh """
          A=1
          echo $A     // 这会读取 Groovy 解释器中的变量 A
      """
      sh '''
          A=1
          echo $A     // 这会读取 shell 中的变量 A
      '''
  }
  ```
- 每个 sh 语句会被 Jenkins 保存为一个临时的 x.sh 文件，用 `/bin/bash -ex x.sh` 的方式执行，且切换到该 Job 的工作目录。
  - 因此各个 sh 语句之间比较独立、解耦。
  - 因此会记录下执行的每条 shell 命令及其输出。例如执行以下 sh 语句：
    ```groovy
    sh """
        echo hello 你好         # 建议不要在 sh 语句中通过 echo 命令添加注释，因为该注释会打印一次，执行命令时又会记录一次，而且命令中包含中文时还会转码
        comment=测试开始：       # 建议通过给变量赋值的方式加入注释
        comment=( 测试开始 )     # 赋值为数组类型，则可以加入空格
    """
    ```
    执行后记录的 Console Output 为：
    ```sh
    +echo hello $'\344\275\240\345\245\275'
    hello 你好
    +comment=测试开始：
    +comment=( 测试开始 )
    ```

### bat

：用于在 Windows 系统上执行 CMD 命令。

### build

：触发一个 Job 。
- 例：
  ```groovy
  build (
      job: 'job1',
      parameters: [
          string(name: 'AGENT', value: 'master'),  // 这里的 string 是指输入值的类型，可输入给大部分类型的 parameters
      ]
  )
  ```
- 一个 Job 可以不指定 agent 、不执行具体命令，只是调用另一个 Job 。

### script

：用于执行 Groovy 代码。
- 可以用赋值号 = 直接创建变量。如下：
  ```groovy
  steps {
      script {
          A = 1
      }
      echo "$A"
  }
  ```
  - 在 script{} 中创建的变量会在 Groovy 解释器中一直存在，因此在该 script{} 甚至该 stage{} 结束之后依然可以读取，但并不会被 Jenkins 加入到 shell 的环境变量中。

- 例：将 shell 命令执行后的 stdout 或返回码赋值给变量
  ```groovy
  script {
      STDOUT = sh(script: 'echo hello', returnStdout: true).trim()
      EXIT_CODE = sh(script: 'echo hello', returnStatus: true).trim()
      echo "$STDOUT"
      echo "$EXIT_CODE"
  }
  ```
  - .trim() 方法用于去掉字符串末尾的空字符、换行符。

- 例：从 shell 中获得数组并遍历它
  ```groovy
  script {
      FILE_LIST = sh(script: "ls /", returnStdout: true)
      for (f in FILE_LIST.tokenize("\n")){
          sh "echo $f"
      }
  }
  ```
  - .tokenize() 方法用于将字符串分割成多个字段的数组，并忽略内容为空的字段。

- 例：捕捉异常
  ```groovy
  script {
      try {
          sh 'exit 1'
      } catch (err) {     // 将异常捕捉之后，构建状态就会依然为 SUCCESS
          echo "${err}"
      } finally {
          echo "finished"
      }
  }
  ```
  - 也可以用 post{} 语句块实现异常处理。

- 例：修改 Job 的描述
  ```groovy
  script {
      currentBuild.rawBuild.project.description = 'Hello'
  }
  ```
  - 执行时可能报错：`RejectedAccessException: Scripts not permitted to use method xxx` \
    需要到 Jenkins 管理页面，点击 `Script Approval` ，批准该方法被脚本调用。

### retry

：用于当某段任务执行失败时（不包括语法错误、超时的情况），自动重试。
- 例：
  ```groovy
      sh 'ls /tmp/f1'
  retry(3) {       // 加上第一次失败的次数，最多执行 3 次
  }
  ```

### timeout

：用于设置超时时间。
- 超时之后则放弃执行，并将任务的状态标记成 ABORTED 。
- 例：
  ```groovy
  timeout(time: 3, unit: 'SECONDS') {     // 单位可以是 SECONDS、MINUTES、HOURS
      sh 'ping baidu.com'
  }
  ```

### waitUntil

：用于暂停执行任务，直到满足特定的条件。
- 例：
  ```groovy
  waitUntil {
      fileExists '/tmp/f1'
  }
  ```

### withEnv

：用于插入环境变量到 sh 语句中。
- 例：
  ```groovy
  withEnv(['A=Hello', 'B=World']) {
        sh 'echo $A $B'
  }
  ```

### withCredentials

：用于调用 Jenkins 的凭据。
- 例：
  ```groovy
  withCredentials([
      usernamePassword(
          credentialsId: 'credential_1',
          usernameVariable: 'USERNAME',   // 将凭据的值存到变量中（如果在终端显示该变量的值，Jenkins 会自动隐藏）
          passwordVariable: 'PASSWORD'
      )]) {
      sh '''                              // 此时 sh 语句需要用单引号，避免票据变量被导入 shell 的环境变量
          docker login -u ${USERNAME} -p ${PASSWORD} ${image_hub}
      '''
  }
  ```

### emailext

：用于发送邮件。
- 需要先在 Jenkins 系统配置中配置 SMTP 服务器。
- 例：
  ```groovy
  emailext (
      subject: "[${currentBuild.fullDisplayName}]的构建结果为${currentBuild.currentResult}",
      to: '123456@email.com',
      from: "Jenkins <123456@email.com>",
      body: """
          任务名：${env.JOB_NAME}
          任务链接：${env.JOB_URL}
          构建编号：${env.BUILD_NUMBER}
          构建链接：${env.BUILD_URL}
          构建耗时：${currentBuild.duration} ms
      """
  )
  ```

### checkout

：用于拉取代码。
- 例：从 Git 仓库拉取代码
  ```groovy
  script {
      checkout([
          $class: 'GitSCM',
          branches: [[name: "$BRANCH"]],                    // 切换到指定的分支，也可以填 tag 或 commit ID
          extensions: [
                        [$class: 'CleanBeforeCheckout'],    // 清理项目文件，默认启用。相当于 git clean -dfx 加 git reset --hard
                        [$class: 'RelativeTargetDirectory', relativeTargetDir: '.'] // 本地仓库的保存目录，默认为 .
                      ],
          userRemoteConfigs: [[
                      credentialsId: "credential_for_git",  // 登录 git 服务器的凭据，为 Username With Password 类型
                      url: "$repository_url"                // git 远程仓库的地址
                      ]]
      ])
  }
  ```

- 例：从 SVN 仓库拉取代码
  ```groovy
  script {
      checkout([
          $class: 'SubversionSCM',
          locations: [[
              remote: "$repository_url"
              credentialsId: 'credential_for_svn',
              local: '.',                               // 本地仓库的保存目录，默认是创建一个与 SVN 最后一段路径同名的子目录
              // depthOption: 'infinity',               // 拉取的目录深度，默认是无限深
          ]],
          quietOperation: true,                         // 不显示拉取代码的过程
          workspaceUpdater: [$class: 'UpdateUpdater']   // 使本地目录更新到最新版本
      ])
  }
  ```

### lock

：用于获取一个全局锁，可避免并发任务同时执行时冲突。
- 可用范围：steps{}、options{}
- 该功能由插件 Lockable Resources 提供。
- 例：
  ```groovy
  lock('resource_1') {    // 锁定一个名为 resource-1 的资源。如果该资源不存在则自动创建（任务结束之后会删除）。如果该资源已经被锁定，则一直等待获取
      sleep 10            // 获取锁之后，执行一些语句
      echo 'done'
  }                       // 执行完之后，会自动释放锁定的资源
  ```
- lock 函数的可用参数如下：
  ```groovy
  lock(resource: 'resource_1',        // 要锁定的资源名
        // label: 'my_resource',      // 通过标签筛选锁定多个资源
        // quantity: 0,               // 至少要锁定的资源数量、默认为 0 ，表示锁定所有
        // variable: 'LOCK',          // 将资源名赋值给一个变量
        // inversePrecedence: false,  // 如果有多个任务在等待获取锁，是否插队到第一个
        // skipIfLocked: false        // 如果需要等待获取锁，是否跳过执行
        ) {
      ...
  }
  ```

## matrix{}

：包含一个 axes{} 和 stages{} ，用于将一个 stages{} 在不同场景下并行执行一次，相当于执行多个实例。
- 可用范围：stage{}
- 每个并行任务称为一个 Branch 。
  - 只要有一个并行执行的任务失败了，最终结果就是 Failure 。
- 例：
  ```groovy
  matrix {
      axes {
          axis {
              name 'PLATFORM'
              values 'linux', 'darwin', 'windows'
          }
          axis {
              name 'PYTHON_VERSION'
              values '3.5', '3.6', '3.7', '3.8'
          }
      }
      stages {
          stage('单元测试') {
              steps {
                  echo PLATFORM
                  echo PYTHON_VERSION
              }
          }
      }
  }
  ```
  - axes{} 用于定义并发任务的矩阵，可以包含多个 axis{} 。
    - 每个 axis{} 用于定义一个矩阵变量。
    - 上例中定义了两个 axis{} ，矩阵变量 PLATFORM、PYTHON_VERSION 分别有 3、4 种取值，因此会执行 3*4=12 个并发任务。


## parallel{}

：包含多个 stage{} ，用于并行执行多个任务。
- 可用范围：stage{}
- 例：
  ```groovy
  stage('单元测试') {
      parallel {
          stage('单元测试 1') {
              steps {
                  echo '测试完成'
              }
          }
          stage('单元测试 2') {
              steps {
                  echo '测试完成'
              }
          }
      }
  }
  ```

## options{}

：用于给 Pipeline 添加一些可选配置。
- 可用范围：pipeline{}、stage{}
- 例：
  ```groovy
  options {
      retry(3)
      timestamps()                        // 输出信息到终端时，加上时间戳
      timeout(time: 60, unit: 'SECONDS')
      disableConcurrentBuilds()           // 不允许同时执行该 job ，会排队执行
      buildDiscarder logRotator(daysToKeepStr: '30', numToKeepStr: '300')  // 限制构建记录保留的最多天数、最大数量，超过限制则删除。这可以限制其占用的磁盘空间，但会导致统计的总构建次数减少
      parallelsAlwaysFailFast()           // 用 matrix{}、parallel{} 执行并发任务时，如果有某个任务失败，则立即放弃执行其它任务
      lock('resource-1')                  // 获取全局锁（此时不支持附加语句块）
  }
  ```

## triggers{}

：用于在满足条件时自动触发 Pipeline 。
- 可用范围：pipeline{}
- 例：
  ```groovy
  triggers {
      cron('H */4 * * 1-5')       // 定期触发。其中 H 表示一个随机值，用于分散执行多个同时触发的任务
      pollSCM('H */4 * * 1-5')    // 定期检查 SCM 仓库，如果提交了新版本代码则构建一次
  }
  ```

## when{}

：用于在满足条件时才执行某个阶段。
- 可用范围：stage{}
- 不满足 when{} 的条件时，会跳过执行该阶段，但并不会导致执行结果为 Failure 。
- 常见的几种定义方式：
  ```groovy
  when {
      environment name: 'A', value: '1'  // 当环境变量等于指定值时
  }
  ```
  ```groovy
  when {
      branch 'dev'            // 当分支为 dev 时（仅适用于多分支流水线）
  }
  ```
  ```groovy
  when {
      expression {            // 当 Groovy 表达式为 true 时
          return params.A
      }
  }
  ```
  ```groovy
  when {
      not {                   // 当子条件为 false 时
          environment name: 'A', value: '1'
      }
  }
  ```
  ```groovy
  when {
      allOf {                 // 当子条件都为 true 时
          environment name: 'A', value: '1'
          environment name: 'B', value: '2'
      }
      branch 'dev'            // 默认就可以包含多个条件，相当于隐式的 allOf{}
  }
  ```
  ```groovy
  when {
      anyOf {                 // 当子条件只要有一个为 true 时
          environment name: 'A', value: '1'
          environment name: 'B', value: '2'
      }
  }
  ```

## input{}

：用于暂停某个阶段的执行，等待用户输入某些参数。
- 可用范围：stage{}
- 例：
  ```groovy
  input {
      message '等待输入...'
      ok '确定'
      submitter 'admin, ops'  // 限制有输入权限的用户
      parameters {            // 等待用户输入以下参数
          string(name: 'NODE', defaultValue: 'master', description: '部署到哪个节点？')
      }
  }
  ```

## post{}

：用于当构建状态满足某些条件时，才执行操作。
- 可用范围：pipeline{}、stage{}
- pipeline 出现语法错误时，Jenkins 会直接报错，而不会执行 post 部分。
- 可用条件：
  ```sh
  success       # 状态为成功
  failure       # 失败
  unstable      # 不稳定
  aborted       # 放弃执行

  changed       # 与上一次执行的状态不同
  regression    # 状态为 failure、unstable 或 aborted ，且上一次执行的状态为 success
  fixed         # 状态为 success ，且上一次执行的状态为 failure 或 unstable

  unsuccessful  # 状态不是 success
  always        # 匹配任何状态
  cleanup       # 匹配任何状态，且放在其它所有 post 条件之后执行，通常用于清理
  ```
- 例：
  ```groovy
  pipeline {
      agent any
      stages {
          stage('Test') {
              steps {
                  echo 'testing ...'
              }
          }
      }
      post {
          success {
              echo '执行成功'
          }
          failure {
              echo '执行失败'
          }
          unstable {
              echo '执行状态不稳定'
          }
          aborted {
              echo '放弃执行'
          }
      }
  }
  ```