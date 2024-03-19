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

        import org.yaml.snakeyaml.Yaml
        import org.yaml.snakeyaml.constructor.Constructor
        
        stage('Remove Services') {
            steps {
                script {
                    def services = readFile('service_update/services.txt').replaceAll('_', '-').split("\n")

                    def yaml = new Yaml(new Constructor())
                    def existingDeployContent = yaml.loadAll(readFile('deployment.yaml'))
                    def existingServiceContent = yaml.loadAll(readFile('service.yaml'))

                    def updatedDeployContent = existingDeployContent.findAll { deploy ->
                        !services.any { service -> deploy.metadata.name == "${service}-deployment" }
                    }

                    def updatedServiceContent = existingServiceContent.findAll { service ->
                        !services.any { service -> service.metadata.name == "${service}-service" }
                    }

                    if (updatedDeployContent.size() != existingDeployContent.size() || updatedServiceContent.size() != existingServiceContent.size()) {
                        writeFile(file: 'deployment.yaml', text: yaml.dumpAll(updatedDeployContent.iterator()))
                        writeFile(file: 'service.yaml', text: yaml.dumpAll(updatedServiceContent.iterator()))
                    } else {
                        println('No services to remove. Skipping this stage.')
                    }
                }
            }
        }

        stage('Add New Services') {
            steps {
                script {
                    def services = readFile('service_update/services.txt').replaceAll('_', '-').split("\n")

                    def existingDeployContent = readFile('deployment.yaml')
                    def existingServiceContent = readFile('service.yaml')

                    def newServices = services.findAll { service ->
                        !existingDeployContent.contains("name: ${service}-deployment") || !existingServiceContent.contains("name: ${service}-service")
                    }

                    // If there are no new services, skip this stage
                    if (newServices.isEmpty()) {
                        println('No new services to add. Skipping this stage.')
                        return
                    }

                    def deployTemplate = readFile('service_update/deployfile.txt')
                    def serviceTemplate = readFile('service_update/servicefile.txt')

                    def usedPorts = (existingDeployContent =~ /containerPort: (\d+)/).collect { it[1].toInteger() }
                    def maxPort = usedPorts ? usedPorts.max() + 1 : 6001

                    newServices.each { service ->
                        maxPort++

                        def formattedServiceName = service.replaceAll('_', '-')

                        if (!existingDeployContent.contains("name: ${formattedServiceName}-deployment")) {
                            def newDeployContent = deployTemplate.replaceAll('name_service', formattedServiceName).replaceAll('number', maxPort.toString()).replaceAll('name_container', service)
                            existingDeployContent += "\n" + newDeployContent
                        }

                        if (!existingServiceContent.contains("name: ${formattedServiceName}-service")) {
                            def newServiceContent = serviceTemplate.replaceAll('name_service', formattedServiceName).replaceAll('number', maxPort.toString())
                            existingServiceContent += "\n" + newServiceContent
                        }
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
                }
            }
        }

        stage('Push the changed deployment file to Git') {
            steps {
                script {
                    sh """
                        git config --global user.name "Nhathuy1305"
                        git config --global user.email "ITITIU20043@student.hcmiu.edu.vn"
                        git add deployment.yaml service.yaml
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