pipeline {

    agent any

    stages {
        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from SCM') {
            steps {
                git branch: 'master', credentialsId: 'github', url: 'https://github.com/Nhathuy1305/Thesis-Report-Management-System-CD.git'
            }
        }

        stage('Update the Deployment Tags') {
            steps {
                script {
                    def services = readFile('services.txt').split("\n")

                    services.each { service ->
                        sh """
                            sed -i 's|daniel135dang/${service}:.*|daniel135dang/${service}:${IMAGE_TAG}|g' deployment.yaml
                        """
                    }
                    sh "cat deployment.yaml"
                }
            }
        }

        stage('Push the changed deployment file to Git') {
            steps {
                script {
                    sh """
                        git config --global user.name "Nhathuy1305"
                        git config --global user.email "ITITIU20043@student.hcmiu.edu.vn"
                        git add deployment.yaml
                        git commit -m "Updated Deployment Manifest"
                    """
                    withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
                        sh "git push https://github.com/Nhathuy1305/Thesis-Report-Management-System-CD.git master"
                    }
                }
            }
        }
    }
}