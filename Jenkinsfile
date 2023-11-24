pipeline {
    agent {label "tbmdcvm02ris01ua" }

    environment {
        oldJarFilePath = "/app/ms/ms-content-data-parser/ms-content-data-parser.jar"
        newJarFilePath = "${WORKSPACE}/target/ms-content-data-parser.jar"
        backupFilename = "/app/ms/ms-content-data-parser/ms-content-data-parser_backup_"
        service_name = "ms-content-data-parser@7402.service"
        microservice = "ms-content-data-parser"
        gitUrl = "https://gitlab.tradebulls.in/trading/vendors/trendlyne/ms-content-data-parser.git"
        branch = "UAT"
    }

    stages {
        stage("Code"){
            steps {
                withCredentials([usernamePassword(credentialsId: "tradebullsgitlab", usernameVariable: "username", passwordVariable: "password")]) {
                    git url: "https://gitlab.tradebulls.in/trading/vendors/trendlyne/ms-content-data-parser.git", branch: "UAT", credentialsId: "tradebullsgitlab"
                    echo "this is the first build"
                }
            }
        }

        stage("Building the code using maven"){
            steps {
                sh 'mvn clean package --settings /home/jenkins/.m2/settings.xml'
                echo "build has been created"
            }
        }

        stage('Need Approval for procced') {
            steps {
                input message: 'Do you want to proceed with deployment?', ok: 'Deploy'
            }
        }

        stage("Taking backup of old JAR file") {
            steps {
                script {
                    if (fileExists(env.oldJarFilePath)) {
                        def timestamp = new Date().format("yyyy-MM-dd_HH:mm:ss")
                        def backupFilename = "${env.backupFilename}${timestamp}.jar"

                        sh "sudo mv ${env.oldJarFilePath} ${backupFilename}"
                        echo "Old JAR file '${env.oldJarFilePath}' has been backed up as '${backupFilename}'"
                    } else {
                        echo "Old JAR file is absent from the folder"
                    }
                }
            }
        }

        stage("Move new JAR file") {
            steps {
                script {
                    sh "sudo mv ${env.newJarFilePath} ${env.oldJarFilePath}"
                    echo "New JAR file '${env.newJarFilePath}' has been moved to '${env.oldJarFilePath}'"
                }
            }
        }

        stage("Changing permission of new JAR file"){
            steps {
                sh "sudo chown app:app ${env.oldJarFilePath}"
            }
        }

        stage('Check services status of ms-backoffice-commoditytrade') {
            steps {
                sh """
                    if systemctl is-active --quiet "${env.service_name}" ; then
                        echo "Service is active, proceeding with the action."
                    else
                        echo "Service is inactive or failed, but we will continue with the action."
                    fi
                    /bin/ps -ef | /bin/grep ${env.microservice}
                """
            }
        }

        stage('Deploy the service'){
            steps {
                sh "sudo systemctl restart ${env.service_name}"
            }
        }

        stage("Status of Deploy service"){
            steps {
                sh """
                systemctl status ${env.service_name}
                /bin/ps -ef | /bin/grep ${env.microservice}
                """
            }
        }
    }
}
