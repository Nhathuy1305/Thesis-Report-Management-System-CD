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

        // stage('Remove Services') {
        //     steps {
        //         script {
        //             def services = readFile('service_update/services.txt').split("\n")

        //             def existingDeployContent = readFile('deployment.yaml').split("\n")
        //             def existingServiceContent = readFile('service.yaml').split("\n")

        //             def existingDeployServices = existingDeployContent.findAll { it =~ /name: (\w+)-deployment/ }.collect { it[1] }
        //             def existingServiceServices = existingServiceContent.findAll { it =~ /name: (\w+)-service/ }.collect { it[1] }

        //             def servicesToRemove = (existingDeployServices + existingServiceServices) - services

        //             [existingDeployContent, existingServiceContent].each { existingContent ->
        //                 servicesToRemove.each { service ->
        //                     def startIndices = []
        //                     existingContent.eachWithIndex { line, index ->
        //                         if (line == '---' && existingContent[index + 2].contains("name: ${service}")) {
        //                             startIndices << index
        //                         }
        //                     }
        //                     def endIndices = startIndices.collect { it + existingContent.subList(it, existingContent.size()).indexOf('---', 1) }

        //                     if (!startIndices.isEmpty() && !endIndices.isEmpty()) {
        //                         for (i in startIndices.size() - 1..0) {
        //                             existingContent = existingContent.subList(0, startIndices[i]) + existingContent.subList(endIndices[i] + 1, existingContent.size())
        //                         }
        //                     }
        //                 }
        //             }

        //             // If there are no services to remove, skip this stage
        //             if (servicesToRemove.isEmpty()) {
        //                 currentBuild.result = 'NOT_BUILT'
        //                 error('No services to remove. Skipping this stage.')
        //             }

        //             writeFile(file: 'deployment.yaml', text: existingDeployContent.join("\n"))
        //             writeFile(file: 'service.yaml', text: existingServiceContent.join("\n"))
        //         }
        //     }
        // }

        stage('Add New Services') {
            steps {
                script {
                    def services = readFile('service_update/services.txt').split("\n")

                    def existingDeployContent = readFile('deployment.yaml')
                    def existingServiceContent = readFile('service.yaml')

                    def newServices = services.findAll { service ->
                        !existingDeployContent.contains("name: ${service}-deployment") && !existingServiceContent.contains("name: ${service}-service")
                    }

                    // If there are no new services, skip this stage
                    if (newServices.isEmpty()) {
                        currentBuild.result = 'NOT_BUILT'
                        error('No new services to add. Skipping this stage.')
                    }

                    def deployTemplate = readFile('service_update/deployfile.txt')
                    def serviceTemplate = readFile('service_update/servicefile.txt')

                    def usedPorts = (existingDeployContent =~ /containerPort: (\d+)/).collect { it[1].toInteger() }
                    def maxPort = usedPorts ? usedPorts.max() : 6000

                    newServices.each { service ->
                        maxPort++

                        def formattedServiceName = service.replaceAll('_', '-')

                        def newDeployContent = deployTemplate.replaceAll('name_service', formattedServiceName).replaceAll('number', maxPort.toString()).replaceAll('name_container', service)
                        def newServiceContent = serviceTemplate.replaceAll('name_service', formattedServiceName).replaceAll('number', maxPort.toString())

                        existingDeployContent += "\n" + newDeployContent
                        existingServiceContent += "\n" + newServiceContent
                    }

                    writeFile(file: 'deployment.yaml', text: existingDeployContent)
                    writeFile(file: 'service.yaml', text: existingServiceContent)
                }
            }
        }

        stage('Update the Deployment Tags') {
            steps {
                script {
                    def services = readFile('service_update/services.txt').split("\n")

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