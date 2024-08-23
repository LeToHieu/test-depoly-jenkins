pipeline {

    agent any

    // tools { 
    //     maven 'my-maven' 
    // }
    tools {
        'org.jenkinsci.plugins.docker.commons.tools.DockerTool' 'docker'

        'maven' 'my-maven' 
    }
    environment {
        MYSQL_ROOT_LOGIN = credentials('mysql-root-login')
    }
    stages {

        stage('Build with Maven') {
            steps {
                // git url: 'https://github.com/cyrille-leclerc/multi-module-maven-project'
                // withMaven {
                //     sh "mvn clean verify"
                // } 
                sh 'mvn --version'
                sh 'docker --version'
                sh 'docker image pull mysql:8.0'
                sh 'java -version'
                sh 'mvn clean package -Dmaven.test.failure.ignore=true'
            }
        }

        stage('Packaging/Pushing images') {
            steps {
                script {
                    //  sh 'docker image pull mysql:8.0'
                //     docker.withTool('docker'){
                        withDockerRegistry(credentialsId: 'DockerHub', url: 'https://index.docker.io/v1/', toolName: 'docker') {
                            sh 'docker build -t khinesss/springboot .'
                            sh 'docker push khinesss/springboot'
                    //     }
                    }
                }
            }
        }

        stage('Deploy MySQL to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull mysql:8.0'
                sh 'docker network create dev || echo "this network exists"'
                sh 'docker container stop khalid-mysql || echo "this container does not exist" '
                sh 'echo y | docker container prune '
                sh 'docker volume rm khalid-mysql-data || echo "no volume"'

                sh "docker run --name khalid-mysql --rm --network dev -v khalid-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_LOGIN_PSW} -e MYSQL_DATABASE=db_example  -d mysql:8.0 "
                sh 'sleep 20'
                sh "docker exec -i khalid-mysql mysql --user=root --password=${MYSQL_ROOT_LOGIN_PSW} < script"
            }
        }

        stage('Deploy Spring Boot to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull khinesss/springboot'
                sh 'docker container stop khalid-springboot || echo "this container does not exist" '
                sh 'docker network create dev || echo "this network exists"'
                sh 'echo y | docker container prune '
                sh 'docker container run -d --rm --name khalid-springboot -p 8081:8080 --network dev khinesss/springboot'
            }
        }
 
    }
    post {
        // Clean after build
        always {
            cleanWs()
        }
    }
}
