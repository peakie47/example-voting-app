pipeline {
    agent none
    
    // Worker start
    stages {
        stage('BuildWorker') {
            agent {
                docker {
                    image 'maven:alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when {
                changeset "**/worker/**"
            }
            steps {
                echo 'Compiling worker app'
                dir('worker'){
                    sh 'mvn compile' 
                }
            }
        }
        stage('TestWorker') {
            agent {
                docker {
                    image 'maven:alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when {
                changeset "**/worker/**"
            }
            steps {
                echo 'Running unit tests on worker app'
                dir('worker'){
                    sh 'mvn clean test' 
                }
            }
        }

        stage('PackageWorker') {
            agent {
                docker {
                    image 'maven:alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when {
                branch 'master'
                changeset "**/worker/**"
            }
            steps {
                echo 'Packaging worker app'
                dir('worker'){
                    sh 'mvn package -DskipTests'
                    archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                }
            }
        }

        stage('docker-packageWorker') {
            agent any
            when {
                branch 'master'
                changeset "**/worker/**"
            }
            steps {
                echo 'Packaging worker app with docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def workerImage = docker.build("peakie47/worker:v${env.BUILD_ID}", "./worker")
                        workerImage.push()
                        workerImage.push("${env.BRANCH_NAME}")
                    }
                }
            }
        }
        // Worker end

        // Result start
        stage('BuildResult') {
            agent {
                docker{
                    image 'node:8.16.0-alpine'
                }
            }
            when {
                changeset "**/result/**"
            }
            steps {
                echo 'Compiling result app'
                dir('result'){
                    sh 'npm install'
                }
            }
        }
        stage('TestResult') {
            agent {
                docker{
                    image 'node:8.16.0-alpine'
                }
            }
            when {
                changeset "**/result/**"
            }
            steps {
                echo 'Running unit tests on result app'
                dir('result'){
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }

        stage('docker-packageResult') {
            agent any
            when {
                branch 'master'
                changeset "**/result/**"
            }
            steps {
                echo 'Packaging result app with docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def resultImage = docker.build("peakie47/result:v${env.BUILD_ID}", "./result")
                        resultImage.push()
                        resultImage.push("${env.BRANCH_NAME}")
                    }
                }
            }
        }
        // Result end


        // Vote start
        stage('BuildVote') {
            agent {
                docker{
                    image 'python:2.7.16-slim'
                    args '-u root -v python_local:/.local'
                }
            }
            when {
                changeset "**/vote/**"
            }
            steps {
                echo 'Compiling vote app'
                dir('vote'){
                    sh 'pip install -r requirements.txt'
                }
            }
        }
        stage('TestVote') {
            agent {
                docker{
                    image 'python:2.7.16-slim'
                    args '-u root -v python_local:/.local'
                }
            }
            when {
                changeset "**/vote/**"
            }
            steps {
                echo 'Running unit tests on vote app'
                dir('vote'){
                    sh 'pip install -r requirements.txt'
                    sh 'nosetests -v'
                }
            }
        }

        stage('docker-packageVote') {
            agent any
            when {
                branch 'master'
                changeset "**/vote/**"
            }
            steps {
                echo 'Packaging vote app with docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def voteImage = docker.build("peakie47/vote:v${env.BUILD_ID}", "./vote")
                        voteImage.push()
                        voteImage.push("${env.BRANCH_NAME}")
                    }
                }
            }
        }
        // Vote end
    }

    post {
        always {
            echo 'Build pipeline for worker is complete'
        }
        failure {
            slackSend (channel: "instavote-cd", message: "Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        success {
            slackSend (channel: "instavote-cd", message: "Build Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }

        always {
            echo 'Build pipeline for result app is complete'
        }
        failure {
            slackSend (channel: "instavote-cd", message: "Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        success {
            slackSend (channel: "instavote-cd", message: "Build Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }

        always {
            echo 'Build pipeline for vote app is complete'
        }
        failure {
            slackSend (channel: "instavote-cd", message: "Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        success {
            slackSend (channel: "instavote-cd", message: "Build Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
    }
}

