pipeline {
    agent {
        docker {
            image 'maven:3.6.0-jdk-11'
        }
    }
    stages {
        stage('Setup') {
            steps {
                echo 'Setup'
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
