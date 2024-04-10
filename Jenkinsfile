pipeline {
    agent {
        node {
            label 'AGENT-1'
        }
    }
    environment { 
        packageVersion = ''
        nexusURL = '172.31.37.174:8081'
    }
    options {
        timeout(time: 1, unit: 'HOURS')
        disableConcurrentBuilds()
    }
    //build
    stages {
        stage('Get the version') { 
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'   
                    // read Json jenkins pipeline, for this install"Pipeline Utility Steps 2.16.1" in jenkins(manage jenkins)  
                    packageVersion = packageJson.version
                    echo "application version: $packageVersion" 
                }
            
            }
        }
         stage('Install dependencies') { 
            steps {
                sh """
                    npm install
                """
            }
        }
        stage('Build') { 
            steps {
                sh """
                    ls -la
                    zip -q -r catalogue.zip ./* -x ".git" -x ".zip"
                    ls -ltr
                """
            }
        }
         stage('Publish Artifact') { 
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${nexusURL}",
                    groupId: 'com.roboshop',
                    version: "${packageVersion}",
                    repository: 'catalogue',
                    credentialsId: 'nexus-auth',
                    artifacts: [
                        [artifactId: 'catalogue',
                        classifier: '',
                        file: 'catalogue.zip',
                        type: 'zip']
                    ]
                )
            }
        }
    
        stage('Deploy') { 
            steps {
                script {
                    def params = [
                        string(name: 'version',value: "$packageVersion"),
                        string(name: 'environment',value: "dev")
                    ]
                    build job: "catalogue-deploy", wait: true, parameters: params
                }
           }
        }
    }
    //post build
    post {
        always {
            echo " I will always say Hello again"
            deleteDir()
        }
        failure { 
            echo 'this runs when pipeline is failed, used generally to send some alerts'
        }
        success{
            echo 'I will say Hello when pipeline is success'
        }
    }
}