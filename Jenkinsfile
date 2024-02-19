pipeline{
    agent any

    environment {
        IMAGE_NAME = "registry.gitlab.com/buaseefah/groupworkjenkins-jenkins"
    }

    stages{
        stage("Set Enviroment") {
            agent {
                label 'vm2-tester'
            }
            steps {
                
                sh 'python3 -m venv myenv'
            }
        }
        stage("Install lib") {
            agent {
                label 'vm2-tester'
            }
            steps {
                sh '''#!/bin/bash
                source myenv/bin/activate && pip install -r ./requirements.txt
                
                '''
            }
        }
        stage("Run Unit Test") {
            agent{
                label 'vm2-tester'
            }
            steps {
                sh '''#!/bin/bash
                source myenv/bin/activate && python3 ./test_restapi.py
                '''
            }
        }
        stage("Create Images") {
            agent {
                label 'vm2-tester'
            }
            steps {
                sh "docker compose build"
                sh "docker ps"
            }
        }
        stage("Create Container") {
            agent{
                label 'vm2-tester'
            }
            steps {
                sh "docker compose up -d"
            }
        }
        stage("Clone Robot") {
            agent{
                label 'vm2-tester'
            }
            steps {
                sh "rm -rf ./assi_jenkins"
                withCredentials([gitUsernamePassword(credentialsId: '79f5bea3-84ff-4a86-9932-bbfd03cb0431', gitToolName: 'git-tool')]) {
                    // Use withCredentials block to securely access credentials
                    sh 'git clone https://gitlab.com/sdp12/assi_jenkins.git'
                }
            }
        }
        stage("Run Robot Test") {
            agent{
                label 'vm2-tester'
            }
            steps {
                // sh "cd ./robottestapi && robot ./plus.robot"
                sh "cd ./assi_jenkins && python3 -m robot ./plus.robot"
                
            }
        }
        stage("Push Image") {
            agent{
                label 'vm2-tester'
            }
            steps{
                    // push the image to the gitlab registry with credentials
                    withCredentials([usernamePassword(credentialsId: '79f5bea3-84ff-4a86-9932-bbfd03cb0431', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh 'docker login -u ${USERNAME} -p ${PASSWORD} registry.gitlab.com'
                        sh 'docker push registry.gitlab.com/buaseefah/groupworkjenkins-jenkins'
                    }
                    sh 'docker rmi -f registry.gitlab.com/buaseefah/groupworkjenkins-jenkins'
                    // sh "docker push registry.gitlab.com/autyauth1/softdevcicd"
            }
        }
        stage('Clean Workspace') {
            agent {
                label 'vm2-tester'
            }
            steps {
                sh 'docker compose -f ./compose.yml down'          
                sh 'docker system prune -a -f'
            }
        }
        stage('Pull Image from Registry') {
            agent {
                label 'vm3-pre-prod'
            }
            steps {
                    withCredentials([usernamePassword(credentialsId: '79f5bea3-84ff-4a86-9932-bbfd03cb0431', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh 'docker login -u ${USERNAME} -p ${PASSWORD} registry.gitlab.com'
                        sh 'docker pull registry.gitlab.com/buaseefah/groupworkjenkins-jenkins'
                    }

            }
        }
        stage('Create Container from Image') {
            agent {
                label 'vm3-pre-prod'
            }
            steps {
                sh "docker compose -f ./compose.yml up -d"
            }
        }
    }
    
}