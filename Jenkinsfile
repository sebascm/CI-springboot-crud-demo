pipeline {
     parameters{
        string(name:'GIT_REPO_APP', 
               defaultValue:
               'https://github.com/losete/springboot-crud-demo.git')
        string(name:'APP_GIT_BRANCH',
               defaultValue: "master")
        string(name:'GIT_USER',
               defaultValue:"sebascm")
        string(name:'REPORT_MAIL',
               defaultValue:"sebastiancalvom@gmail.com")
        choice(name: 'AUTOMERGE_BRANCH',
                choices: ['Yes', 'No'],
                description: 'Automatic merge into \'dev\' of the branch above in case the build is successful')   
        string(name:'MYSQL_DB_PASSWORD',
               defaultValue:"rootpass")
        string(name:'MYSQL_DB_NAME',
               defaultValue:"springbootdb") 
    }
    agent { docker { image 'chusca/docker-and-maven-3.6.0-jdk-11:latest' } }
    stages {
        stage('Setup') {
            steps {
                git branch: "${params.APP_GIT_BRANCH}",
                    credentialsId: "${params.GIT_USER}",
                    url: "${params.GIT_REPO_APP}"
                sh 'mkdir reports' 
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
            steps{
                git branch: "${params.APP_GIT_BRANCH}", credentialsId: "${params.GIT_USER}", url: "${params.GIT_REPO_APP}"
                script {
                    try {
                        sh "docker run --name mysql-${BUILD_ID}  -e MYSQL_ROOT_PASSWORD=${MYSQL_DB_PASSWORD} -e MYSQL_DATABASE=${MYSQL_DB_NAME} -d mysql"
                        sh 'mkdir -p web/src/test/resources/'
                        sh 'echo "spring.datasource.url=jdbc:mysql://172.17.0.8:3306/${MYSQL_DB_NAME}" > web/src/test/resources/application.properties'
                        sh 'echo "spring.datasource.username=root" >> web/src/test/resources/application.properties'
                        sh 'echo "spring.datasource.password=${MYSQL_DB_PASSWORD}" >> web/src/test/resources/application.properties'
                        sh 'echo "spring.jpa.hibernate.ddl-auto=update" >> web/src/test/resources/application.properties'
                        sh 'mvn test'
                    } catch (Exception e) {
                        sh 'Handle the exception!'
                    } finally {
                        sh "docker stop mysql-${BUILD_ID}"
                        sh "docker rm mysql-${BUILD_ID}"
                    }
                }
            }
        }
        stage ("Launch testing pipeline"){
            steps {
                script{
                    if ("${params.APP_GIT_BRANCH}" == 'master')){
                        sh 'tar -cvzf project.tar.gz project'
                        archiveArtifacts artifacts: 'project.tar.gz', fingerprint: true

                        build job: 'spring-boot-crud-demo-Testing',
                            propagate: true,
                            wait: true,
                            parameters: [[$class: 'StringParameterValue',
                                        name: 'APP_GIT_BRANCH',
                                        value: "${APP_GIT_BRANCH}"]
                }
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
            archiveArtifacts artifacts'reports.tar.gz', fingerprint: true
            emailext (attachmentsPattern: 'reports.tar.gz',
                body: "Workflow result on ${currentBuild.currentResult}, check attached artifacts for further information",
                subject: "Jenkins Build ${currentBuild.currentResult} on Job ${env.JOB_NAME}",
                from: 'notificaciones.torusnewies@gmail.com',
                replyTo: '',
                to: "${params.REPORT_MAIL}"
            )
        }
        failure {
            archiveArtifacts artifacts: 'reports.tar.gz', fingerprint: true
        }
        cleanup{
            cleanWs()
        }
    }
}
