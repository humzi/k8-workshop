pipeline {
    agent any

    environment {
        REGISTRY    = 'local-registry:5000'
        IMAGE       = 'demo-app'
        // the repo that holds BOTH the app source and the deploy manifests (same repo here)
        GIT_HOST    = 'github.com/humzi/k8-workshop.git'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm    // clones the app repo (the one containing this Jenkinsfile)
            }
        }

        stage('Build image') {
            steps {
                sh '''
                  cd demo-app
                  sudo podman build -t ${REGISTRY}/${IMAGE}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Push image') {
            steps {
                sh 'sudo podman push --tls-verify=false ${REGISTRY}/${IMAGE}:${BUILD_NUMBER}'
            }
        }

        stage('Update config repo') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-pat',
                                 usernameVariable: 'GIT_USER',
                                 passwordVariable: 'GIT_TOKEN')]) {
                    sh '''
                      rm -rf config
                      git clone https://${GIT_USER}:${GIT_TOKEN}@${GIT_HOST} config
                      cd config/deploy
                      sed -i "s|image: .*/demo-app:.*|image: ${REGISTRY}/${IMAGE}:${BUILD_NUMBER}|" demo-app.yaml
                      git config user.email "jenkins@local"
                      git config user.name "Jenkins CI"
                      git commit -am "ci: deploy demo-app build ${BUILD_NUMBER}" || echo "no change to commit"
                      git push origin HEAD:main
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Build ${BUILD_NUMBER} pushed and config repo updated — ArgoCD will sync."
        }
        failure {
            echo "Pipeline failed — check which stage above went red."
        }
    }
}
