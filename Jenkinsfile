def ls_command
pipeline {
    agent {
        label 'master'
    }
    stages {
        stage('checkout'){
            input{
                message 'Enter Host'
                    id 'Host'
                    ok 'OK'
                    parameters {
                        string defaultValue: 'Host', description: '', name: 'TARGET', trim: false
                    }
                
            steps {
                script {
                    git credentialsId: 'devopint', url: 'https://github.com/DevOpsINT/Course.git'
                }
            }
        }
        stage('shell command example') {
            steps {
                script {
                    sh "sudo su"
                    sh "ansible-playbook -i "$TARGET," -u ubuntu -b --private-key=my.pem playbook.yaml"
                }
            }
        }
    }
}
