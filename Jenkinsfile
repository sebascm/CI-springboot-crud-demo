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
        choice(name: 'AUTOMERGE_BRANCH',
                choices: ['Yes', 'No'],
                description: 'Automatic merge into \'dev\' of the branch above in case the build is successful')   
    }
    agent { docker { image 'maven:3.6.0-jdk-11' } }
    options {
        copyArtifactPermission('**')
    }
    stages {
        stage('Setup') {
            steps {
                git branch: "${params.APP_GIT_BRANCH}",
                    credentialsId: "${params.GIT_USER}",
                    url: "${params.GIT_REPO_APP}"
                sh 'mkdir reports' 
            }
        }
        stage('DB-setup') {
            steps {
                echo 'DB-setup'
            }
        }
        stage('Validate') {
            steps {
                sh 'mvn validate'
                sh 'ls -la target'
                sh 'mv target/checkstyle-result.xml reports'
            }
        }
        stage('Compile') {
            steps {
                sh 'mvn compile  > reports/compile.txt'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test -Dmaven.test.failure.ignore=true  > reports/tests.txt'
            }
        }
        stage('Spotbugs') {
            steps {
                sh 'mvn verify -Dmaven.test.failure.ignore=true > reports/bugs.txt'
            }
        }
        stage('Benchmark') {
            steps {
                script {
                    try {
                        timeout(time: 10, unit: 'MINUTES') {
                            sh 'mvn package  -Dmaven.test.failure.ignore=true '
                            sh 'java -jar benchmarks/target/benchmarks.jar > reports/benchmarks.txt'
                        }
                    } catch (err) {
                        // This try catch prevents Jenkins from setting currentBuild to
                        // ABORTED in case of benchmark failure
                        writeFile(file: "reports/benchmark_report.txt",
                                text: "Benchmarks are too slow", encoding: "UTF-8")
                    }
                }
            }
        }
        stage('Spring-boot:run') {
            steps {
                sh 'mvn package  -Dmaven.test.failure.ignore=true '
                //sh 'java -jar benchmarks/target/benchmarks.jar'
            }
        }
    }
    post {
        success{
            script{
                if (params.AUTOMERGE_BRANCH == 'Yes'){
                    withCredentials([usernamePassword(credentialsId: 'SebasGH', passwordVariable: 'pass', usernameVariable: 'user')]) {
                        sh "git config --global user.email 'sebastiancalvom@gmail.com'"
                        sh "git config --global user.name $user" //may be are multiple sebas in a development team
                        sh "git remote update"
                        sh "git fetch --all"
                        sh "git pull --all"
                        sh "git checkout dev"
                        sh "git merge origin/master"
                        sh "git merge ${BRANCH_NAME}"
                        sh "git push https://$user:$pass@github.com/losete/springboot-crud-demo"
                    }
                }
            }
        }
        always {
            sh 'tar -cvzf reports.tar.gz reports/'
            archiveArtifacts 'reports.tar.gz'
            emailext (attachmentsPattern: 'reports.tar.gz',
                body: "Workflow result on ${currentBuild.currentResult}, check attached artifacts for further information",
                subject: "Jenkins Build ${currentBuild.currentResult} on Job ${env.JOB_NAME}",
                from: 'notificaciones.torusnewies@gmail.com',
                replyTo: '',
                to: "${params.REPORT_MAIL}"
            )
        }
        cleanup{
            cleanWs()
        }
    }
}
