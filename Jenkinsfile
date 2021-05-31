pipeline {
    agent any

    stages {
        stage('Prepare') {
            steps {
                echo "1.Prepare Stage"
                checkout scm
                script {
                    build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
                    if (env.BRANCH_NAME != 'master') {
                        build_tag = "${env.BRANCH_NAME}-${build_tag}"
                    }
                }
            }
        }
        stage('Test') {
            steps {
                echo "2.Test Stage"
            }
        }
        stage('Build') {
            steps {
                echo "3.Build Docker Image Stage"
                sh "docker build -t vctvg/kbtest:${build_tag} ."
            }
        }
        stage('Push') {
            steps {
                echo "4.Push Docker Image Stage"
                withCredentials([usernamePassword(credentialsId: 'Registry', passwordVariable: 'RegistryPassword', usernameVariable: 'RegistryUser')]) {
                    sh "docker login -u ${RegistryUser} -p ${RegistryPassword}"
                    sh "docker push vctvg/kbtest:${build_tag}"
                }
            }
        }
        stage('Deploy') {
            steps {
                echo "5. Deploy Stage"
                script {
                    if (env.BRANCH_NAME == 'master') {
                        input "Confirm？"
                    }
                }
                sh "sed -i 's/<BUILD_TAG>/${build_tag}/' dep.yml"
                sh "sed -i 's/<BRANCH_NAME>/${env.BRANCH_NAME}/' dep.yml"
                sh "kubectl apply -f dep.yml --record"
            }   
        }
    }
}

