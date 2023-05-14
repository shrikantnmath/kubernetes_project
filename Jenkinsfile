node {
    
    stage('Git checkout'){
        git 'https://github.com/shrikantnmath/kubernetes_project.git'
    }
    
    stage('sending docker file to Ansible server over ssh'){
        sshagent(['ansible-demo']) {
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.45.51'
            sh 'scp /var/lib/jenkins/workspace/pipeline-demo/* ubuntu@172.31.45.51:/home/ubuntu'
      }
    }
    stage('Docker Build image'){
        sshagent(['ansible-demo']) {
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.45.51 cd /home/ubuntu/'
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.45.51 docker image build -t $JOB_NAME:v1.$BUILD_ID .'
        }
    }
    stage('Docker image tagging'){
        sshagent(['ansible-demo']) {
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.45.51 cd /home/ubuntu/'
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.45.51 docker image tag $JOB_NAME:v1.$BUILD_ID  shrikantnmath/$JOB_NAME:v1.$BUILD_ID '
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.45.51 docker image tag $JOB_NAME:v1.$BUILD_ID  shrikantnmath/$JOB_NAME:latest '
        }   
    
    }
    stage('push docker image to docker hub'){
        sshagent(['ansible-demo']) {
            withCredentials([string(credentialsId: 'dockerhub_passwd', variable: '')]) {    
                 sh "ssh -o StrictHostKeyChecking=no ubuntu@172.31.45.51 docker login -u shrikantnmath -p ${dockerhub_passwd}"
                 sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.45.51 docker image push shrikantnmath/$JOB_NAME:v1.$BUILD_ID '
                 sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.45.51 docker image push shrikantnmath/$JOB_NAME:latest '
            }
        }
    }
    stage('copy files from ansible to kubernetes server'){
        ssh(['kubernetes_server']) {
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.10.21'
            sh 'scp /var/lib/jenkins/workspace/pipeline-demo/* ubuntu@172.31.10.21:/home/ubuntu'
        }
    }
    stage('kubernetes deployment using ansible'){
        sshagent(['ansible-demo']) {
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.45.51'
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.45.51 ansible-playbook ansible.yml'
        }
    }
}

