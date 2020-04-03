---
title: 持续集成、自动化部署
date: 2020-01-16 12:33:56
categories:
- 笔记
tags: 
- CI
---


自动化部署流程

<!-- more -->
### 代码提交
```flow
st=>start: Developer
op_dev=>operation: Gitlab dev branch
op_rel=>operation: Gitlab rel branch
cond_merge=>condition: Merge request
cond_test=>condition: Test
e=>end: Gitlab master branch
st->op_dev(right)->cond_merge
cond_merge(yes)->op_rel
cond_merge(no)->op_dev
op_rel->cond_test
cond_test(yes)->e
cond_test(no)->op_rel
```


开发提交代码到dev分支，自测通过后提merge request合并到release分支，交由测试人员测试，测试通过合并至master分支，等待上线。建议每次大版本升级打上tag，方便追溯。hotfix和feature暂不使用。

### 编译、打包、备份、部署
```flow
st=>start: Jenkins Master
op_git=>operation: Gitlab
op_jenkins=>operation: Jenkins Node
op_backup=>operation: Backup Server
cond_build=>condition: Build Success
e=>end: Deploy Server
st->op_git->op_jenkins(right)->cond_build
cond_build(yes)->op_backup
cond_build(no)->op_git
op_backup->e
```
Jenkins通过Jenkinsfile分配节点进行编译、打包、备份、部署。

### Jenkinsfile示例

java1.0自动化打包docker镜像及部署脚本示例：
<font color=blue>这个脚本包含了所有的模块，实际使用中可以把Jenkinsfile进行拆分，每个模块单独写一个Jenkinsfile，待优化。</font>
```groovy
properties([
    buildDiscarder(
        logRotator(
            artifactDaysToKeepStr: '', 
            artifactNumToKeepStr: '', 
            daysToKeepStr: '', 
            numToKeepStr: '10'
        )
    ),
    parameters([
        gitParameter(branch: '',
                     branchFilter: 'origin/(.*)',
                     defaultValue: 'master',
                     description: '',
                     name: 'BRANCH',
                     quickFilterEnabled: false,
                     selectedValue: 'NONE',
                     sortMode: 'NONE',
                     tagFilter: '*',
                     type: 'PT_BRANCH')
    ])

def compileProject(pom, jar){
    try {
        withMaven() {
            echo "====++++start compile++++===="
            sh "mvn -f ${pom} clean install"
            // archiveArtifacts jar
        }
    }catch (err){
        echo "Failed: ${err}"
        error "maven package failed"
        }
}

def buildImage(jar, dockerfile, module, commit_ID){     
    echo "====++++get latest commitID++++===="
    echo "====++++commitID:${commit_ID}++++===="
    sh "cp ${jar} ${dockerfile}"
    dir(dockerfile) {
        echo "====++++build docker images++++===="
        try {
            sh """
                docker build -t xxx.xxx.x.xxx/pipeline/${module}:${commit_ID} .
                docker tag xxx.xxx.x.xxx/pipeline/${module}:${commit_ID} xxx.xxx.x.xxx/pipeline/${module}:pre
            """
        }catch(err) {
            echo "Failed: ${err}"
            error "build docker image failed"
        }
    }
}

def pushImage(module, commit_ID){
    echo "====++++push docker images++++===="
    try{
        //TODO:docker golbal variable
        withDockerRegistry(credentialsId: 'credentialsId', url: 'https://xxx.xxx.x.xxx') {
            sh "docker push xxx.xxx.x.xxx/pipeline/${module}:${commit_ID}"
            sh "docker push xxx.xxx.x.xxx/pipeline/${module}:pre"
        }
    }catch (err){
        echo "Failed: ${err}"
        error "push docker image failed"
    }
}

def cleanImages(module, commit_ID){
    echo "====++++clean local docker images++++===="
    try{
        sh "docker rmi xxx.xxx.x.xxx/pipeline/${module}:${commit_ID}"
        sh "docker rmi xxx.xxx.x.xxx/pipeline/${module}:pre"
    }catch(err){
        echo "Failed:${err}"
        error "clean local docker images failed"
    }
}

node("CI_SAAS") {

    currentBuild.description = params.MODULE

    def moduleName = params.MODULE
    def pomDir = pomMap.get(moduleName)
    def dockerfileDir
    def jarDir

    stage('chechout'){
        echo "====++++start checkout++++===="
        echo "gitUrl:${env.GITURL}"
        echo "branch:${params.BRANCH}"
        echo "pomDir:${pomDir}"
        echo "jarDir:${jarDir}"
        git branch: params.BRANCH, credentialsId: 'credentialsId', url: env.GITURL
    }

    def commitID = sh(returnStdout: true, script: "git rev-parse --short HEAD").trim()

    stage("compile") {
        compileProject(pomDir,jarDir)
    }

    if('all'.equals(moduleName)){
        for(keyName in jarMap.keySet()){
            jarDir = jarMap.get(keyName)
            dockerfileDir = dockerfileMap.get(keyName)
            println(jarDir+":"+dockerfileDir)
            archiveArtifacts jarDir
            // stage("compile") {
            //     compileProject(pomDir,jarDir)
            // }
            stage("build docker image"){
                buildImage(jarDir, dockerfileDir, keyName, commitID)
            }
            stage("push docker image"){
                pushImage(keyName, commitID)
            }
            stage("clean local docker images"){
                cleanImages(keyName, commitID)
            }
        }
    }else if('common'.equals(moduleName)){
        archiveArtifacts jarDir
        // stage("compile") {
        //         compileProject(pomDir,jarDir)
        //     }
    }else{ 
        dockerfileDir = dockerfileMap.get(moduleName)
        jarDir = jarMap.get(moduleName)
        archiveArtifacts jarDir
        // stage("compile") {
        //     compileProject(pomDir,jarDir)
        // }
        stage("build docker image"){
            buildImage(jarDir, dockerfileDir, moduleName, commitID)
        }
        stage("push docker image"){
            pushImage(moduleName, commitID)
        }
        stage("clean local docker images"){
            cleanImages(moduleName, commitID)
        }
    }
    

    stage("compile") {
        try {
            withMaven() {
                echo "====++++start compile++++===="
                sh "mvn -f ${params.POMDIR} clean package"
                archiveArtifacts params.JARDIR
            }
        }catch (err){
            echo "Failed: ${err}"
            error "maven package failed"
        }
    }

    if (!"common".equals(params.MODULE) && !"all".equals(params.MODULE)){
        stage("build docker image") {
            echo "====++++get latest commitID++++===="
            echo "====++++commitID:${commitID}++++===="
            sh "cp ${params.JARDIR} ${params.DOCKERFILEDIR}"
            dir(params.DOCKERFILEDIR) {
                echo "====++++build docker images++++===="
                try {
                    sh """
                        docker build -t xxx.xxx.x.xxx/pipeline/${params.MODULE}:${commitID} .
                        docker tag xxx.xxx.x.xxx/pipeline/${params.MODULE}:${commitID} xxx.xxx.x.xxx/pipeline/${params.MODULE}:pre
                    """
                }catch(err) {
                    echo "Failed: ${err}"
                    error "build docker image failed"
                }
            }
        }

        stage("push docker image") {
            echo "====++++push docker images++++===="
            try{
                //TODO:docker golbal variable
                withDockerRegistry(credentialsId: 'credentialsId', url: 'https://xxx.xxx.x.xxx') {
                    sh "docker push xxx.xxx.x.xxx/pipeline/${params.MODULE}:${commitID}"
                    sh "docker push xxx.xxx.x.xxx/pipeline/${params.MODULE}:pre"
                }
            }catch (err){
                echo "Failed: ${err}"
                error "push docker image failed"
            }
        }

        node("172.16.1.113") {
            sh "docker stop ${params.MODULE}"
            sh "docker rm ${params.MODULE}"
            try {
                withDockerRegistry(credentialsId: 'credentialsId', url: 'https://xxx.xxx.x.xxx') {
                    sh "docker run -d --cpus=2 --restart=always --network=host --name=${params.MODULE} \
                        -v /home/apps/logs:/home/apps/logs \
                        xxx.xxx.x.xxx/planb_pre/registeration:pre \
                    "
                }
            }catch (err){

                echo "Failed: ${err}"
                error "run docker image failed"
            }
        }
    }
    
}
```
