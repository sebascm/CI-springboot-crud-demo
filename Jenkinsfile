pipeline {
     parameters{
        string(name:'GIT_REPO_APP',
               defaultValue:
               'https://github.com/losete/springboot-crud-demo.git')
        string(name:'APP_GIT_BRANCH',
               defaultValue: "**")
        string(name:'GIT_USER',
               defaultValue:"sebascm")
        string(name:'REPORT_MAIL',
               defaultValue:"sebastiancalvom@gmail.com")
    }
    agent none
    stages {
        stage('Setup') {
            agent {docker 'sebascm/mysql-standalone'}
            steps {
                git branch: "${params.APP_GIT_BRANCH}",
                    credentialsId: "${params.GIT_USER}",
                    url: "${params.GIT_REPO_APP}"
                sh 'rm -R reports'
                sh 'mkdir reports' 
                sh 'ls -la'
            }
        }
        stage('Validate') {
            agent {docker 'maven:3.6.0-jdk-11'}
            steps {
                sh 'mvn validate'
                sh 'ls -la target'
                sh 'mv target/checkstyle-result.xml reports'
            }
        }
        stage('Compile') {
            agent {docker 'maven:3.6.0-jdk-11'}
            steps {
                sh 'mvn compile  > reports/compile.txt'
            }
        }
        stage('Test') {
            agent {docker 'maven:3.6.0-jdk-11'}
            steps {
                sh 'mvn test -Dmaven.test.failure.ignore=true  > reports/tests.txt'
            }
        }
        stage('Spotbugs') {
            agent {docker 'maven:3.6.0-jdk-11'}
            steps {
                sh 'mvn verify -Dmaven.test.failure.ignore=true > reports/bugs.txt'
            }
        }
        stage('Benchmark') {
            agent {docker 'maven:3.6.0-jdk-11'}
            steps {
                script {
                    try {
                        timeout(time: 1, unit: 'MINUTES') {
                            echo 'Test'
                        }
                    } catch (err) {
                        // This try catch prevents Jenkins from setting currentBuild to
                        // ABORTED in case of benchmark failure
                        writeFile(file: "reports/benchmark_report.txt",
                                text: "Benchmarks too slow", encoding: "UTF-8")
                    }
                }
            }
        }
        stage('Spring-boot:run') {
            agent {docker 'maven:3.6.0-jdk-11'}
            steps {
                 script {
                    if (${params.APP_GIT_BRANCH} == 'master'){
                        echo 'Spring'
                    }
                }
            }
        }
    }
    post {
        always {
            sh 'tar -cvzf reports.tar.gz reports'
            emailext (attachmentsPattern: 'reports.tar.gz',
                body: "Workflow result on ${currentBuild.currentResult}, check attached artifacts for further information",
                subject: "Jenkins Build ${currentBuild.currentResult} on Job ${env.JOB_NAME}",
                from: 'notificaciones.torusnewies@gmail.com',
                replyTo: '',
                to: "${params.REPORT_MAIL}"
            )
            archiveArtifacts artifacts: 'reports.tar.gz', fingerprint: true
        }
        failure {
            echo 'failure'
        }
        cleanup{
            cleanWs()
        }
    }
}
