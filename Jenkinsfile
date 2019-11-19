pipeline {
     parameters{
        string(name:'GIT_REPO_APP',
               defaultValue:
               'https://github.com/losete/springboot-crud-demo.git')
        string(name:'APP_GIT_BRANCH',
               defaultValue: "**")
        string(name:'GIT_USER',
               defaultValue:"sebascm")   
    }
    agent {
        docker {
            image 'maven:3.6.0-jdk-11'
        }
    }
    stages {
        stage('Setup') {
            steps {
                git branch: "${params.APP_GIT_BRANCH}",
                    credentialsId: "${params.GIT_USER}",
                    url: "${params.GIT_REPO_APP}"
                sh 'mkdir reports' 
                sh 'ls -la'
            }
        }
        stage('Validate') {
            steps {
                echo 'Validate'
            }
        }
        stage('Compile') {
            steps {
                echo 'Compile'
            }
        }
        stage('Test') {
            steps {
                echo 'Test'
            }
        }
        stage('Spotbugs') {
            steps {
                echo 'Spotbugs'
            }
        }
        stage('Benchmark') {
            steps {
                echo 'Benchmark'
            }
        }
        stage('Spring-boot:run') {
            steps {
                echo 'Spring-boot:run'
            }
        }
    }
    post {
        always {
            echo 'always'
        }
        failure {
            echo 'failure'
        }
        cleanup{
            cleanWs()
        }
    }
}
