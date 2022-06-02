pipeline {
  agent any
  stages {
    stage('pull project') {
      steps {
        echo '准备拉取代码'
        checkout([$class: 'GitSCM', branches: [[name: '*/${branch}']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/gongjixiaobai/jenkins.git']]])
        echo '代码拉取成功'
      }
    }

    stage('build project准备') {
      steps {
        echo '准备打包项目'
        sh 'mvn clean package -Dmaven.test.skip=true'
        echo '项目打包成功'
        sh 'pwd'
        echo '项目地址'
      }
    }

    stage('删除容器') {
      steps {
        sh '''containerId=`docker ps -a | grep javaapplication | awk \'{print $1}\'`
                echo "容器id : " ${containerId}
                if [[ -n ${containerId} ]]
                  then
                    docker rm -f ${containerId}
                    echo "容器已退出"
                  else
                    echo "暂无启动容器"
                fi
                '''
      }
    }

    stage('docker制作镜像') {
      steps {
        sh '''contanerId=`docker images | grep -w javaapplication | grep -v / | awk \'{print $3}\'`
                cd /home/projects
                rm -rf *.jar
                cp /root/.jenkins/workspace/java_pipeline/target/*.jar /home/projects
                if [[ -n ${contanerId} ]]
                  then 
                    docker rmi -f ${contanerId}
                    echo "镜像已删除"
                  else
                    echo "暂无该镜像"
                fi
                
                cat << EOF > Dockerfile
                FROM java:8
                
                WORKDIR /home
                
                ENV JAVA_OPTS="-Xmx500m -Xms500m"
                
                COPY entrypoint.sh entrypoint.sh
                
                RUN chmod 755 entrypoint.sh
                
                COPY javaApplication.jar javaApplication.jar
                
                ENTRYPOINT ["./entrypoint.sh"]
                
                
                '''
      }
    }

    stage('制作docker-compose.yml') {
      steps {
        sh '''
                cd /home/projects
                docker build -t javaapplication:1 .
                echo "镜像制作成功"
                cat << EOF > docker-compose.yml
                version: "3"
                services: 
                  java: 
                    build: .
                    container_name: javaapplication
                    image: javaapplication:1
                    ports: 
                      - 8083:8083
                    volumes:
                      - /home/projects:/home:rw
                    environment: 
                      - JAVA_OPTS=-Xms512m -Xmx512m -XX:+UseParallelOldGC
                '''
      }
    }

    stage('使用docker-compose发布镜像') {
      steps {
        sh '''
                cd /home/projects
                docker-compose up -d
                
                runId=`docker ps | grep javaapplication | awk \'{print $1}\'`
                
                if [[ -n ${runId} ]]
                  then 
                    echo "容器启动成功"
                  else
                    echo "容器启动失败"
                fi'''
      }
    }

  }
}
