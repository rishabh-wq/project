//pipeline code
pipeline {
    agent any
    stages {
        stage('Networking Configuration') {
            steps {
                sh 'docker network rm project_project || true'
                sh 'docker container rm $(docker container ls -aq) || true' 
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'cd $WORKSPACE'
                git branch: "master",
                    url: "https://github.com/mayur321886/project"
                sh 'ls'
            }
        }
        post {
            any {
                sh 'rm -rf project || true' 
            }
        }
        stage('Pre-Build Tests') {
            parallel {
                stage('Git Repository Scanner'){
                    steps {
                        sh 'cd $WORKSPACE'
                        sh 'trufflehog https://github.com/mayur321886/project --json | jq "{branch:.branch, commitHash:.commitHash, path:.path, stringsFound:.stringsFound}" > trufflehog_report.json || true'
                        sh 'cat trufflehog_report.json'
                        sh 'echo "Scanning Repositories.....done"'
                    }
                }
                post {
                    success {
                        archiveArtifacts artifacts: 'trufflehog_report.json', onlyIfSuccessful: true
                        emailext attachLog: true, attachmentsPattern: 'trufflehog_report.json', 
                        body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}\n Thankyou,\n CDAC-Project Group-7", 
                        subject: "Jenkins Build ${currentBuild.currentResult}: Job ${env.JOB_NAME} - success", mimeType: 'text/html', to: "mayur321886@gmail.com"
                    }
                }
                post {
                    failure {
                        archiveArtifacts artifacts: 'trufflehog_report.json', onlyIfSuccessful: true
                        emailext attachLog: true, attachmentsPattern: 'trufflehog_report.json', 
                        body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}\n Thankyou,\n CDAC-Project Group-7", 
                        subject: "Jenkins Build ${currentBuild.currentResult}: Job ${env.JOB_NAME} - failure", mimeType: 'text/html', to: "mayur321886@gmail.com"
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
                        archiveArtifacts artifacts: '*.json'
                    }
                }
                post {
                    success {
                        archiveArtifacts artifacts: '*.json', onlyIfSuccessful: true
                        emailext attachLog: true, attachmentsPattern: '*.json', 
                        body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}\n Please Find Attachments for the following:\n Thankyou\n CDAC-Project Group-7",
                        subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - success", mimeType: 'text/html', to: "mayur321886@gmail.com"
                    }
                }
                post {
                    failure {
                        archiveArtifacts artifacts: '*.json', onlyIfSuccessful: true
                        emailext attachLog: true, attachmentsPattern: '*.json', 
                        body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}\n Please Find Attachments for the following:\n Thankyou\n CDAC-Project Group-7",
                        subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - failure", mimeType: 'text/html', to: "mayur321886@gmail.com" 
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
                sh 'docker run --name login  --network project_project -p 80:8080 -d prod_tomcat' 
            }
        }
        stage('SonarQube Analysis') {
            steps {
                sh ' || true'
            }
        }
        stage('SCA') {
            parallel {
                stage('Dependency Check') {
                    steps {
                        sh 'wget https://github.com/mayur321886/project/blob/master/dc.sh'
                        sh 'chmod +x dc.sh'
                        sh './dc.sh'
                    }
                }
                post {
                    success {
                        archiveArtifacts artifacts: 'odc-reports/*.html', onlyIfSuccessful: true
                        archiveArtifacts artifacts: 'odc-reports/*.csv', onlyIfSuccessful: true
                        archiveArtifacts artifacts: 'odc-reports/*.json', onlyIfSuccessful: true
                        emailext attachLog: true, attachmentsPattern: '*.html', 
                        body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}\n Please Find Attachments for the following:\n Thankyou\n CDAC-Project Group-7",
                        subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - success", mimeType: 'text/html', to: "mayur321886@gmail.com"
                    }
                }
                post {
                    failure {
                        archiveArtifacts artifacts: 'odc-reports/*.html', onlyIfSuccessful: true
                        archiveArtifacts artifacts: 'odc-reports/*.csv', onlyIfSuccessful: true
                        archiveArtifacts artifacts: 'odc-reports/*.json', onlyIfSuccessful: true
                        emailext attachLog: true, attachmentsPattern: '*.html', 
                        body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}\n Please Find Attachments for the following:\n Thankyou\n CDAC-Project Group-7",
                        subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - failure", mimeType: 'text/html', to: "mayur321886@gmail.com"
                    }
                }
                stage('Junit Testing') {
                    steps {
                        sh 'echo "Junit Reports are created using archiveArtifacts"'
                    }
                }
                post {
                    success {
                        archiveArtifacts artifacts: '*junit.xml', onlyIfSuccessful: true
                        emailext attachLog: true, attachmentsPattern: '*junit.xml', 
                        body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}\n Please Find Attachments for the following:\n Thankyou\n CDAC-Project Group-7",
                        subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - success", mimeType: 'text/html', to: "mayur321886@gmail.com"
                    }
                }
                post {
                    failure {
                        archiveArtifacts artifacts: '*junit.xml', onlyIfSuccessful: true
                        emailext attachLog: true, attachmentsPattern: '*junit.xml', 
                        body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}\n Please Find Attachments for the following:\n Thankyou\n CDAC-Project Group-7",
                        subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - failure", mimeType: 'text/html', to: "mayur321886@gmail.com"
                    }
                }
            }
        }
        stage('DAST') {
            steps {
                sh 'docker run --name dast_full --network project_project -t owasp/zap2docker-stable zap-full-scan.py -t http://mayur.cdac.project.com/LoginWebApp/ || true'
                sh 'docker run --name dast_baseline --network project_project -t owasp/zap2docker-stable zap-baseline.py -t http://mayur.cdac.project.com/LoginWebApp/ --autooff || true'
            }
        }
        post {
            success {
                sh 'docker rm dast_full'
                sh 'docker rm dast_baseline'
            }
        }
        post {
            failure {
                sh 'docker rm dast_baseline'
                sh 'docker rm dast_full'
            }
        }
    }
}
