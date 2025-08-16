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
        GITHUB_CREDS = credentials('github-credentials')
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
                    docker.build("alaamohamed1/java_app", "v${BUILD_NUMBER}")
                }
            }
        }
        stage('Push java app image') {
            steps {
                script {
                    def docker = new com.iti.docker()
                    docker.login("${DOCKER_USER}", "${DOCKER_PASS}")
                    docker.push("alaamohamed1/java_app", "v${BUILD_NUMBER}")
                }
            }
        }
        stage('Update ArgoCD repo') {
            steps {
                dir('argocd') {
                    
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: 'main']],
                        doGenerateSubmoduleConfigurations: false,
                        extensions: [],
                        submoduleCfg: [],
                        userRemoteConfigs: [[
                            url: 'https://github.com/AlaaMohamed09/k8s-project.git',
                            credentialsId: 'github-credentials'
                        ]]
                    ])

                    sh """
                        sed -i 's|image: .*|image: alaamohamed1/java_app:v${BUILD_NUMBER}|' ./deployment.yaml
                    """

                    withCredentials([usernamePassword(credentialsId: 'github-credentials', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                       
                        sh """
                            git checkout -B main
                            git config user.email "jenkins@ci.local"
                            git config user.name "Jenkins CI"
                            git add deployment.yaml
                            git commit -m "Update image tag to alaamohamed1/java_app:v${BUILD_NUMBER}" || echo "No changes to commit"
                            git push https://${GIT_USER}:${GIT_PASS}@github.com/AlaaMohamed09/k8s-project.git main
                        """
                    }
                }
            }
        }
    }
}
