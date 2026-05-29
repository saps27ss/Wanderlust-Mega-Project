pipeline {
agent any

```
environment {
    SONAR_HOME = tool "sonar"
    DOCKER_REPO = "swapnilshende27"
}

parameters {
    string(name: 'FRONTEND_DOCKER_TAG', defaultValue: 'latest', description: 'Frontend Docker Tag')
    string(name: 'BACKEND_DOCKER_TAG', defaultValue: 'latest', description: 'Backend Docker Tag')
}

stages {

    stage('Validate Parameters') {
        steps {
            script {
                if (!params.FRONTEND_DOCKER_TAG?.trim() || !params.BACKEND_DOCKER_TAG?.trim()) {
                    error("FRONTEND_DOCKER_TAG and BACKEND_DOCKER_TAG must be provided.")
                }
            }
        }
    }

    stage('Workspace Cleanup') {
        steps {
            cleanWs()
        }
    }

    stage('Git Checkout') {
        steps {
            git branch: 'main',
                credentialsId: 'github',
                url: 'https://github.com/saps27ss/Wanderlust-Mega-Project'
        }
    }

    stage('Trivy Filesystem Scan') {
        steps {
            sh '''
            trivy fs . \
            --severity HIGH,CRITICAL \
            --no-progress
            '''
        }
    }

    stage('SonarQube Analysis') {
        steps {
            withSonarQubeEnv('Sonar') {
                sh '''
                ${SONAR_HOME}/bin/sonar-scanner \
                  -Dsonar.projectKey=wanderlust \
                  -Dsonar.projectName=wanderlust \
                  -Dsonar.sources=.
                '''
            }
        }
    }

    stage('SonarQube Quality Gate') {
        steps {
            timeout(time: 5, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
            }
        }
    }

    stage('Backend Environment Setup') {
        steps {
            dir('Automations') {
                sh '''
                chmod +x updatebackendnew.sh
                ./updatebackendnew.sh
                '''
            }
        }
    }

    stage('Frontend Environment Setup') {
        steps {
            dir('Automations') {
                sh '''
                chmod +x updatefrontendnew.sh
                ./updatefrontendnew.sh
                '''
            }
        }
    }

    stage('Build Backend Image') {
        steps {
            dir('backend') {
                sh """
                docker build \
                -t ${DOCKER_REPO}/wanderlust-backend-beta:${params.BACKEND_DOCKER_TAG} .
                """
            }
        }
    }

    stage('Build Frontend Image') {
        steps {
            dir('frontend') {
                sh """
                docker build \
                -t ${DOCKER_REPO}/wanderlust-frontend-beta:${params.FRONTEND_DOCKER_TAG} .
                """
            }
        }
    }

    stage('Docker Login') {
        steps {
            withCredentials([
                usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )
            ]) {
                sh '''
                echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                '''
            }
        }
    }

    stage('Push Backend Image') {
        steps {
            sh """
            docker push ${DOCKER_REPO}/wanderlust-backend-beta:${params.BACKEND_DOCKER_TAG}
            """
        }
    }

    stage('Push Frontend Image') {
        steps {
            sh """
            docker push ${DOCKER_REPO}/wanderlust-frontend-beta:${params.FRONTEND_DOCKER_TAG}
            """
        }
    }
}

post {
    success {
        build job: 'Wanderlust-CD',
            parameters: [
                string(name: 'FRONTEND_DOCKER_TAG', value: params.FRONTEND_DOCKER_TAG),
                string(name: 'BACKEND_DOCKER_TAG', value: params.BACKEND_DOCKER_TAG)
            ]
    }
}
```

}

