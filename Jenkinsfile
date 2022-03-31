//pipeline code
pipeline {
    agent any
    stages {
        stage('Install Dependencies') {
            steps {
                sh 'cd $WORKSPACE'
                sh 'rm -rf project || true'
                git branch: "master",
                    url: "https://github.com/mayur321886/project"
                sh 'ls'
            }
        }
        stage('Pre-Build Tests') {
            parallel {
                stage('Git Repository Scanner'){
                    steps {
                        sh 'cd $WORKSPACE'
                        sh 'trufflehog https://github.com/mayur321886/project --json | jq "{branch:.branch, commitHash:.commitHash, path:.path, stringsFound:.stringsFound}" > trufflehog_report.json || true'
                        archiveArtifacts artifacts: 'trufflehog_report.json'
                        sh 'cat trufflehog_report.json'
                        sh 'echo "Scanning Repositories.....done"'
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
                sh 'docker stop sonar_for_tomcat && docker rm sonar_for_tomcat || true'
                sh 'docker-compose up -d'
                sh 'docker build -t prod_tomcat .'
                sh 'docker run --name login  --network project_project -p 80:8080 -d prod_tomcat'
                sh 'docker run --name sonar_for_tomcat --network project_project -p 4444:9000 -d owasp/sonarqube'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                sh 'mvn sonar:sonar -Dsonar.projectKey=cdac -Dsonar.host.url=http://mayur.cdac.project.com:4444 -Dsonar.login=147f99ddd003e1a86dbcf3805256cec665c80aed || true'
            }
        }
        stage('SCA') {
            parallel {
                stage('Dependency Check') {
                    steps {
                        sh 'wget https://github.com/mayur321886/project/blob/master/dc.sh'
                        sh 'chmod +x dc.sh'
                        sh './dc.sh'
                        archive (includes: 'dependency-check-report.html')
                        archive (includes: 'dependency-check-report.json')
                        archive (includes: 'dependency-check-report.csv')
                    }
                }
                stage('Junit Testing') {
                    steps {
                        archive (includes: 'dependency-check-junit.xml')
                    }
                }
            }
        }
        stage('DAST') {
            steps {
                sh 'docker rm dast_full || true'
                sh 'docker rm dast_baseline || true'
                sh 'docker run --name dast_full --network project_project -t owasp/zap2docker-stable zap-full-scan.py -t http://mayur.cdac.project.com/LoginWebApp/ || true'
                sh 'docker run --name dast_baseline --network project_project -t owasp/zap2docker-stable zap-baseline.py -t http://mayur.cdac.project.com/LoginWebApp/ --autooff || true'
            }
        }
    }
}
