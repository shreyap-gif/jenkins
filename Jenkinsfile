pipeline {
    agent { label 'ubuntu-agent' }

    stages {
        stage('checkout') {
            steps {
                git url: 'https://github.com/artisantek/jenkins.git'
            }
        }

        stage('Test') {
            when { branch 'dev' }
            steps {
                dir('javaapp-pipeline') {
                    sh 'mvn clean test'
                }
            }
        }

        stage('Build') {
            when { anyOf { branch 'main'; branch 'master' } }
            steps {
                dir('javaapp-pipeline') {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('sonar-analysis') {
            when { anyOf { branch 'main'; branch 'master' } }
            steps {
                withSonarQubeEnv('sonar') {
                    dir('javaapp-pipeline') {
                        sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=test -Dsonar.projectName=test'
                    }
                }
            }
        }

        stage('Trivy-Scan') {
            when { anyOf { branch 'main'; branch 'master' } }
            steps {
                dir('javaapp-pipeline') {
                    sh '''
                    wget -q -O html.tpl https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl
                    trivy fs . --format template --template "@html.tpl" -o report.html
                    '''
                }
            }
        }

        stage('Deploy') {
            when { anyOf { branch 'main'; branch 'master' } }
            steps {
                dir('javaapp-pipeline/target') {
                    sh '''
                    if pgrep -f "java -jar java-sample-21-1.0.0.jar" > /dev/null; then
                        pkill -f "java -jar java-sample-21-1.0.0.jar"
                    fi
                    JENKINS_NODE_COOKIE=dontKillMe nohup java -jar java-sample-21-1.0.0.jar > app.log 2>&1 &
                    '''
                }
            }
        }
    }
}
