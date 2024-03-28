properties([
    parameters([        
        string(name: 'branch', defaultValue: 'main', description: 'branch to build')
    ])
])

pipeline {
   agent {
        kubernetes {
            cloud 'kubernetes'
            label 'mypod'
            yaml """
                apiVersion: v1
                kind: Pod
                spec:
                  containers:
                    - name: jdk17
                      image: openjdk:17-jdk
                      command: ['cat']
                      tty: true
            """
        }
    }
    stages {     
        stage('Clone') {
            steps {
                git credentialsId: 'scocks', url: 'git@github.com:scocks/plataforma-bom.git', branch: "${params.branch}"
            }
        }   
        stage('Generate Properties') {
            steps {
                container('jdk17') {                    
                    withCredentials([usernamePassword(credentialsId: 'nexus-admin-cred', passwordVariable: 'repoPassword', usernameVariable: 'repoUser')]) {
                       sh """
                       echo "repoUser=${repoUser}" > gradle.properties
                       echo "repoPassword=${repoPassword}" >> gradle.properties
                       """
                    }
                }
            }
        }       
        stage('Build and Test') {
            steps {
                container('jdk17') {                                        
                    sh """
                    ./gradlew clean build
                    """                    
                }
            }
        }
        stage('Publish') {
            steps {
                container('jdk17') {                    
                    sh """
                    ./gradlew publish
                    """
                }
            }
        }
    }
}
