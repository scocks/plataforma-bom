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
                    - name: my-java-app
                      image: openjdk:17-jdk
                      command: ['cat']
                      tty: true
            """
        }
    }
    stages {
        stage('Clone Repo') {
            steps {
                git (url : "git@github.com:scocks/plataforma-bom.git", credentialsId : "scocks", branch : "main")
            }
        }
        stage('Build and Test') {
            steps {
                container('my-java-app') {                    
                    withCredentials([usernamePassword(credentialsId: 'nexus-admin-cred', passwordVariable: 'repoPassword', usernameVariable: 'repoUser')]) {
                       sh """
                       echo "repoUser=${repoUser}" > gradle.properties
                       echo "repoPassword=${repoPassword}" >> gradle.properties
                       ./gradlew clean build publish
                       """
                    }
                }
            }
        }
    }
}
