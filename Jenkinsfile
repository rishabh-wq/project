//pipeline code
pipeline {
    agent any
    stages {
        stage('Networking Configuration') {
            steps {
                sh 'docker network rm prod_project || true'
                sh 'docker container rm $(docker container ls -aq) || true' 
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'cd $WORKSPACE'
                sh 'rm -rf project'
                git branch: "master",
                    url: "https://github.com/rishabh-wq/project"
                sh 'ls'
            }
        }
        stage('Pre-Build Tests') {
            parallel {
                stage('Git Repository Scanner') {
                    steps {
                        sh 'cd $WORKSPACE'
                        sh 'trufflehog https://github.com/mayur321886/project --json | jq "{branch:.branch, commitHash:.commitHash, path:.path, stringsFound:.stringsFound}" > trufflehog_report.json || true'
                        sh 'cat trufflehog_report.json'
                        sh 'echo "Scanning Repositories.....done"'
                        archiveArtifacts artifacts: 'trufflehog_report.json', onlyIfSuccessful: true
                        emailext attachLog: true, attachmentsPattern: 'trufflehog_report.json', 
                        body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}\n Thankyou,\n CDAC-Project Group-7", 
                        subject: "Jenkins Build ${currentBuild.currentResult}: Job ${env.JOB_NAME} - success", mimeType: 'text/html', to: "mayur321886@gmail.com"
                    }
                }
                stage('Image Security') {
                    steps {
                        sh 'cd $WORKSPACE'
                        sh 'dockle --input ~/docker_img_backup/mytomcat.tar -f json -o mytomcat_report.json'
                        sh 'cat mytomcat_report.json | jq {summary}'
                        sh 'dockle --input ~/docker_img_backup/pgadmin4.tar -f json -o pgadmin4_report.json'
                        sh 'cat pgadmin4_report.json | jq {summary}'
                        sh 'dockle --input ~/docker_img_backup/postgres11.tar -f json -o postgres11_report.json'
                        sh 'cat postgres11_report.json | jq {summary}'
                        sh 'dockle --input ~/docker_img_backup/zap2docker-stable.tar -f json -o zap2docker-stable_report.json'
                        sh 'cat zap2docker-stable_report.json | jq {summary}'
                        sh 'dockle --input ~/docker_img_backup/sonarqube.tar -f json -o sonarqube_report.json'
                        sh 'cat sonarqube_report.json | jq {summary}'
                        archiveArtifacts artifacts: '*.json', onlyIfSuccessful: true
                        emailext attachLog: true, attachmentsPattern: '*.json', 
                        body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}\n Please Find Attachments for the following:\n Thankyou\n CDAC-Project Group-7",
                        subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - success", mimeType: 'text/html', to: "mayur321886@gmail.com"
                    }
                }
            }
        }
        stage('Build Stage') {
            steps {
                sh 'mvn clean'
                sh 'mvn compile'
                sh 'mvn install package'
            }
        }
        stage('Initializing Docker') {
            steps {
                sh 'docker stop postgres_container && docker rm postgres_container || true'
                sh 'docker stop login || true'
                sh 'docker rm login || true'
                sh 'docker stop pgadmin && docker rm pgadmin || true'
                sh 'docker-compose up -d'
                sh 'docker build -t prod_tomcat .'
                sh 'docker run --name login  --network prod_project -p 80:8080 -d prod_tomcat' 
            }
        }
        stage('SonarQube Analysis') {
            steps {
                sh 'mvn sonar:sonar -Dsonar.projectKey=cdac -Dsonar.host.url=http://192.168.10.137:4444 -Dsonar.login=dcce02acbf9019cdc6d338f50287f8178016fd6d || true'
            }
        }
        stage('SCA') {
            parallel {
                stage('Dependency Check') {
                    steps {
                        sh 'wget https://github.com/rishabh-wq/project/blob/master/dc.sh'
                        sh 'chmod +x dc.sh'
                        sh './dc.sh'
                        archiveArtifacts artifacts: 'odc-reports/*.html', onlyIfSuccessful: true
                        archiveArtifacts artifacts: 'odc-reports/*.csv', onlyIfSuccessful: true
                        archiveArtifacts artifacts: 'odc-reports/*.json', onlyIfSuccessful: true
                        emailext attachLog: true, attachmentsPattern: '*.html', 
                        body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}\n Please Find Attachments for the following:\n Thankyou\n CDAC-Project Group-7",
                        subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - success", mimeType: 'text/html', to: "mayur321886@gmail.com"
                    }
                }
                stage('Junit Testing') {
                    steps {
                        sh 'echo "Junit Reports are created using archiveArtifacts"'
                        archiveArtifacts artifacts: '*junit.xml', onlyIfSuccessful: true
                        emailext attachLog: true, attachmentsPattern: '*junit.xml', 
                        body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}\n Please Find Attachments for the following:\n Thankyou\n CDAC-Project Group-7",
                        subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - success", mimeType: 'text/html', to: "mayur321886@gmail.com"
                    }
                }
            }
        }
        stage('DAST') {
            steps {
                sh 'docker rm dast_baseline || true'
                sh 'docker rm dast_full || true'
                sh 'docker run --name dast_full --network prod_project -t owasp/zap2docker-stable zap-full-scan.py -t http://rishabh.cdac.project.com/LoginWebApp/ || true'
                sh 'docker run --name dast_baseline --network prod_project -t owasp/zap2docker-stable zap-baseline.py -t http://rishabh.cdac.project.com/LoginWebApp/ --autooff || true'
            }
        }
    }
}
