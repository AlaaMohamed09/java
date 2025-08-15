@Library('iti-sharedlib') _
pipeline {
    agent any
    options {
        disableConcurrentBuilds()
    }
    tools {
        jdk 'java-11'
        maven 'mvn-3-5-4'
    }
    environment {
        DOCKER_USER = credentials('docker-username')
        DOCKER_PASS = credentials('docker-password')
    }
    stages {
        stage('Get code') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/master']],
                    extensions: [],
                    userRemoteConfigs: [[url: 'https://github.com/AlaaMohamed09/java.git']]
                )
            }
        }
        stage('Build app') {
            steps {
                sh 'java -version'
                sh 'mvn -version'
                script {
                    def mavenBuild = new org.iti.mvn()
                    mavenBuild.javaBuild("package install")
                }
            }
        }
        stage('Archive app') {
            steps {
                archiveArtifacts artifacts: '**/*.jar', followSymlinks: false
            }
        }
        stage('Docker build') {
            steps {
                script {
                    def docker = new com.iti.docker()
                    docker.build("iti-java", "${BUILD_NUMBER}")
                }
            }
        }
        stage('Push java app image') {
            steps {
                script {
                    def docker = new com.iti.docker()
                    docker.login("${DOCKER_USER}", "${DOCKER_PASS}")
                    docker.push("iti-java", "${BUILD_NUMBER}")
                }
            }
        }
        
  //      stage('Update ArgoCD repo') {
  //          steps {
  //              sh 'mkdir argocd'
  //              dir('argocd') {
  //                  checkout scmGit(
  //                      branches: [[name: '*/main']],
 //                       extensions: [],
 //                       userRemoteConfigs: [[url: 'https://github.com/AlaaMohamed09/k8s-project.git']]
 //                   )
 //                   sh "sed -i 's|image: .*|image: iti-java:${BUILD_NUMBER}|' ./deployment.yaml"
 //               }
 //           }
 //       }
        
    }
}
