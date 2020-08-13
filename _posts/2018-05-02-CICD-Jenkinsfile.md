---
layout: post
title: "CI环境搭建"
categories:
  - CI&CD
---
## CI框架
Jenkins+Gerrit+GIT
### Jenkins与Gerrit的集成
在Gerrit中的配置:  
1.Create the profile through in Gerrit web interface for your Jenkins user, and set up a SSH key for that user.  
2.Gerrit web interface > Admin > Groups > Non-Interactive Users > Add your jenkins user.  
3.Admin > Projects > ... > Access > Edit  
    Reference: refs/*  
    Read: ALLOW for Non-Interactive Users  
    Reference: refs/heads/*  
    Label Code-Review: -1, +1 for Non-Interactive Users  
    Label Verified: -1, +1 for Non-Interactive Users  

IMPORTANT: On Gerrit 2.7+, you also need to grant "Stream Events" capability. Without this, the plugin will not work, will try to connect to Gerrit repeatedly, and will eventually cause OutOfMemoryError on Gerrit.  

1.Gerrit web interface > People > Create New Group : "Event Streaming Users". Add your jenkins user.  
2.Admin > Projects > All-Projects > Access > Edit  
    Global Capabilities  
    Stream Events: ALLOW for Event Streaming Users  
在Jenkins中的配置：  
在Jenkins中的Manage Jenkins-->Plugin Manager中添加Gerrit Trigger插件  
插件安装完后在Manage Jenkins-->Gerrit Trigger中进行设置：  
![Gerrit Trigger设置](https://wiki.jenkins.io/download/attachments/45481993/server-settings.png?version=1&modificationDate=1278331124000&api=v2)
#### jenkins每次都checkout master分支代码的问题
由于Jenkins默认的设置是每次checkout 的代码都是master分支上的代码，因此pachset create时执行的tox测试的代码，并不是本次commit push的代码。需要进行如下设置：  
To get the Git Plugin to download your change; set Refspec to $GERRIT_REFSPEC and the Choosing strategy to Gerrit Trigger. This may be under ''Additional Behaviours/Strategy For Choosing What To Build' rather than directly visible as depicted in the screenshot. You may also need to set 'Branches to build' to $GERRIT_BRANCH. If this does not work for you set Refspec to refs/changes/*:refs/changes/* and 'Branches to build' to $GERRIT_REFSPEC.

Note: Be aware that $GERRIT_BRANCH and $GERRIT_REFSPEC are not set in the Ref Updated case. If you want to trigger a build, you can set Refspec and 'Branches to build' to $GERRIT_REFNAME.
![](https://wiki.jenkins.io/download/attachments/45481993/git_config.png?version=1&modificationDate=1278331124000&api=v2)
### Jenkinsfile编写
可以说jenkinsfile就是pipeline，Jenkinsfile支持Declarative和Scripted Pipelines。Declarative是在2.X版本后开始支持的：
- 提供了更加丰富语法特性
- 更加容易编写和阅读

```groovy
#!groovy
pipeline {
    agent any
    environment {
        DOCKER_IMAGE = sh returnStdout:true, script: "cat Buildfile.yaml|grep docker-image|awk '{printf \$2}'"
        VERSION = sh returnStdout:true, script: 'cat Buildfile.yaml|grep version|awk \'{print $2}\''
    }
    stages {
        stage('unitTest: pep8') {
            steps {
                sh 'tox -e pep8'
            }
        }
        stage('unitTest: py27') {
            steps {
                sh 'tox -e py27'
            }
        }
        stage('Building') {
            when {
                beforeAgent true
                anyOf {
                    changelog 'RPM BUILD'
                    changelog 'DEPLOY'
                }
            }
            agent{
               docker {
                   image "ironic-p"
               }
           }
           steps {
               echo 'Building......'
                sh "git checkout $VERSION"
                sh 'python setup.py sdist'
                sh 'cp dist/*.tar.gz /root/rpmbuild/SOURCES/'
                sh "sed -i \"s/Release:\\([ \\t]*\\)\\([0-9]*\\)\\(.*\$\\)/Release:\\1\${BUILD_ID}\\3/g\" /root/rpmbuild/SPECS/openstack-ironic.spec"
                sh 'rpmbuild -ba /root/rpmbuild/SPECS/openstack-ironic.spec'
                sh "ls -l /root/rpmbuild/RPMS"
                sh "pwd"
                sh "ls"
                sh "cat Buildfile.yaml"
            }
       }
       stage('Deploy') {
            when {
                beforeAgent true
                changelog 'DEPLOY'
            }
           steps {
               echo 'Deploying......'
               ansiblePlaybook disableHostKeyChecking: true, playbook: 'test-deploy.yaml'

           }
       }
    }
}
```

### Jenkinsfile中根据git tag作为判断条件
Jenkinsfile的官方文档中对when的说明中有如下描述：  
tag  
Execute the stage if the <font color=red>TAG_NAME</font> variable matches the given pattern. Example: <font color=red>when { tag "release-*" }</font>. If an empty pattern is provided the stage will execute if the <font color=red>TAG_NAME</font> variable exists (same as <font color=red>buildingTag()</font>).
写法如下：  
```Groovy
when {
    beforeAgent true
    tag comparator: 'REGEXP', pattern: '[0-9]+\\.[0-9]+\\.[0-9]+\\-rc'
}
```
但这里没有生效，显示TAG_NAME为null。  
解决方法：  
后安装插件[Git Parameter Plug-In](https://wiki.jenkins.io/display/JENKINS/Git+Parameter+Plugin),添加TAG_NAME参数：
![BuildByParameter](https://github.com/ronysun/MarkdownImage/raw/master/Jenkins-build-by-paramiter.png)

### environment通过shell输出进行变量赋值，字符串后带有换行符的问题：
```Groovy
    environment {
        TAG_NAME = sh returnStdout:true, script: "git describe --tags"
    }
```
上面的步骤会导致TAG_NAME这个变量的最后有一个看不到的换行符，可能会导致某些问题。
解决方法：
用awk解决：git describe --tags|awk '{printf $1}'
### pipeline脚本中引用环境变量时变量后不能直接带.
```Groovy
       stage('uploadRPM') {
           steps {
               sh "echo hello-$TAG_NAME-$BUILD_ID.EL7"
           }
       }
```
上面的写法会报错，应修改如下：
${BUILD_ID}.EL7
### Jenkins中job运行在docker中时，报错：
![error-build-in-docker](https://github.com/ronysun/MarkdownImage/raw/master/Jenkins-build-in-docker-error.png)
问题原因：[oci-mount的bug导致](https://access.redhat.com/errata/RHBA-2018:0434)  
解决方法：  
升级oci-mount到2.3.3以上版本
升级docker
