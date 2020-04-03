---
title: Jenkins 集成 Hygieia
date: 2019-03-25 18:43:03
categories:
- 笔记
tags: 
- jenkins
- docker-compose
- hygieia
---


docker-compose 搭建 Hygieia 环境

<!-- more -->

## docker-compose.yml
```yaml
version: "2"
services:
  mongodb:
    image: mongo:latest
    environment:
    - MONGODB_USERNAME=dashoarduser
    - MONGODB_DATABASE=dashboarddb
    - MONGODB_PASSWORD=dbpassword
    ports:
    - "$Port:27017"

  hygieia-api:
    image: hygieia-api:latest
    volumes:
    - ./logs:/hygieia/logs
    - ./conf:/hygieia/config
    environment:
    - jasypt.encryptor.password=hygieiasecret
    - SPRING_DATA_MONGODB_DATABASE=dashboarddb
    - SPRING_DATA_MONGODB_HOST=mongodb
    - SPRING_DATA_MONGODB_PORT=27017
    - SPRING_DATA_MONGODB_USERNAME=dashoarduser
    - SPRING_DATA_MONGODB_PASSWORD=dbpassword
    - FEATURE_DYNAMIC_PIPELINE=enabled
    - AUTH_EXPIRATION_TIME=3600000
    - AUTH_SECRET=secret
    - SKIP_PROPERTIES_BUILDER=false
    ports:
      - "$Port:8080"
    links:
    - mongodb

  hygieia-ui:
    image: hygieia-ui:latest
    container_name: hygieia-ui
    environment:
    - API_HOST=hygieia-api
    - API_PORT=8080
    ports:
    - "$Port:80"
    links:
    - hygieia-api

  hygieia-jenkins-build-collector:
    image: hygieia-jenkins-build-collector:latest
    container_name: hygieia-jenkins-build
    volumes:
    - ./logs:/hygieia/logs
    - ./conf:/hygieia/config
    links:
    - mongodb:mongo
    - hygieia-api
    environment:
    - JENKINS_CRON=0 */15 * * * *
    - JENKINS_MASTER=$Jenkins_URL
    - SKIP_PROPERTIES_BUILDER=false
    - MONGO_PORT=27017
```

1.mongodb
```bash
$ docker-compose up -d mongodb
$ docker exec -it mongodb bash
$ mongo 127.0.0.1/admin
$ use dashboarddb
$ db.createUser({user: "dashoarduser", pwd: "dbpassword", roles: [{role: "readWrite", db: "dashboarddb"}]})
```

2.API
```bash
$ docker-compose up -d hygieia-api
```

3.UI
```bash
$ docker-compose up -d hygieia-ui
```

4.hygieia-jenkins-build-collector
```bash
$ docker-compose up -d hygieia-jenkins-build-collector
```

## Jenkins 安装 Hygieia 插件

上传hygieia-publisher.hpi到Jenkins

## Pipeline script

```groovy
node("master"){
    stage("checkout"){
        checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/Sonlan/springboot-demo.git']]])
        sh "ls -al"
        sh "pwd"
    }
    stage("package"){
            sh "pwd"
            sh "ls -al"
            withMaven (maven: 'mvn') {
                sh "mvn -v"
                sh "mvn package -Dmaven.test.skip=true"
            }
        }
    sh 'java -version'
    sh 'git --version'
    stage("hygieia"){
        hygieiaBuildPublishStep buildStatus: 'Success'
        hygieiaArtifactPublishStep artifactDirectory: 'target', artifactGroup: 'com.syna.ci', artifactName: 'springboot-demo*.jar', artifactVersion: ''
        hygieiaDeployPublishStep applicationName: 'SpringBoot', artifactDirectory: 'target', artifactGroup: 'com.syna.ci', artifactName: 'springboot-demo*.jar', artifactVersion: '', buildStatus: 'Success', environmentName: 'Dev'
    }
}
```
