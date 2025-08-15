@Library('iti-sharedlib')_

properties([
    disableConcurrentBuilds()
])

node {

    def javaHome = tool name: 'java-11', type: 'jdk'
    def mavenHome = tool name: 'mvn-3-5-4', type: 'maven'
    def DOCKER_USER = credentials('docker-username')
    def DOCKER_PASS = credentials('docker-password')

    stage("Get code"){
        checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/AlaaMohamed09/java.git']])
    }
    stage("build app"){
        env.JAVA_HOME = javaHome
        env.PATH = "${javaHome}/bin:${mavenHome}/bin:${env.PATH}"
        sh 'java -version'
        sh 'mvn -version'

        def mavenBuild = new org.iti.mvn()
        mavenBuild.javaBuild("package install")

    }
    stage("archive app"){
        archiveArtifacts artifacts: '**/*.jar', followSymlinks: false
    }
    stage("docker build"){
        def docker = new com.iti.docker()
        docker.build("iti-java", "${BUILD_NUMBER}")
    }
    stage("push java app image"){
        def docker = new com.iti.docker()
        docker.login("${DOCKER_USER}", "${DOCKER_PASS}")
        docker.push("iti-java", "${BUILD_NUMBER}")
    }
    stage("push java app image"){
        sh "mkdir argocd"
        sh "cd argocd"
        checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/AlaaMohamed09/k8s-project.git']])
        sh "sed -i #        image: .*#        image: iti-java:${BUILD_NUMBER}# ./deployment.yaml"
    }
}
