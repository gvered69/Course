counter = 1
template_ip = "IP_ADDR_"
hosts_file_name = "hosts"
folder_for_run_playbook = "ansible"
list_of_choices = []
CurrentVersion = 1
def Commit_Id
def image_name
def Build_version
def NextVersion
def  docker_hub_repo
docker_slave_node = 'docker-host'
script_name = "Experiment.py"
test_script_name = "ExperimentTests.py"
folder_scripts= "scripts"
underscore = '_'
colon = ':'
pipeline_failure_indicator_string = 'FAILURE'
dockerize_string = "dockerize"
module="python3"
base_image_name="python:3"
image_name = "python_image:1.0"
pipeline {
    options {
        timeout(8)
    }
    agent {
        label 'master'
    }
    stages {
        stage('checkout') {
            steps {
                script {

                    git branch: 'Lidor', credentialsId: 'dcacc4ee-0dab-4a7f-b781-ad2b883bfe68', url: 'https://github.com/DevOpsINT/Course.git'
                    CurrentVersion = sh(script: "git tag", returnStdout: true).trim()
                    Commit_Id = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    Build_version = CurrentVersion + '_' + Commit_Id
                    NextVersion = CurrentVersion + 1
                }
            }
        }
        stage('Input') {
            steps {
                script {
                    show_dir_content_command = "ls ./ansible/roles"
                    change_first_char_word_command = "sed -E \'s/./\\u&/\'"
                    choices = input(
                            id: 'userInput', message: 'Choose options:', ok: 'Ok', cancel: 'Cancel',
                            parameters: [
                                    booleanParam(defaultValue: false, name: sh(script: "$show_dir_content_command | awk NR==${counter++} | $change_first_char_word_command", returnStdout: true).trim()),
                                    booleanParam(defaultValue: false, name: sh(script: "$show_dir_content_command | awk NR==${counter++} | $change_first_char_word_command", returnStdout: true).trim()),
                                    booleanParam(defaultValue: false, name: sh(script: "$show_dir_content_command | awk NR==${counter++} | $change_first_char_word_command", returnStdout: true).trim()),
                                    booleanParam(defaultValue: false, name: sh(script: "$show_dir_content_command | awk NR==${counter++} | $change_first_char_word_command", returnStdout: true).trim()),
                                    booleanParam(defaultValue: false, name: sh(script: "$show_dir_content_command | awk NR==${counter++} | $change_first_char_word_command", returnStdout: true).trim())

                            ])

                    for (entry in choices) {
                        if (entry.value == true) {
                            list_of_choices.add(entry.key.toLowerCase())
                        }
                    }
                    list_of_choices.sort()
                    dir(folder_for_run_playbook) {

                        sh "cp $hosts_file_name ${hosts_file_name}.bak"
                        for (value in list_of_choices) {
                            def ip = input(
                                    id: 'userInput', message: 'Enter Host info for installing\n ' + value, ok: 'Ok',
                                    parameters: [

                                            string(defaultValue: '192.168.1.3',
                                                    name: 'private_ip'),
                                    ])
                            sh "sed -i 's/\\$template_ip$value\\>/$ip/' $hosts_file_name"
                        }
                    }
                }
            }
        }
        stage('UT'){

            when{
                expression { dockerize_string in list_of_choices }
            }
            steps{
                script{
                    sh "echo Checking the build version: $Build_version"
                    dir('./scripts'){
                        try {
                            sh "python3 $test_script_name"
                        }
                        catch (exception){
                            println "The test is failed"
                            currentBuild.result =  pipeline_failure_indicator_string
                            throw exception
                        }
                    }

                }
            }
        }
        stage('Build docker image'){

            when{
                expression { dockerize_string in list_of_choices }
            }
            steps{
                script{
                    archive_name = 'archive'
                        dir(workspace){
                            stash includes: '', name: archive_name
                        }
                    node(docker_slave_node){
                        try{
                             unstash archive_name
                             sh "sudo docker build --build-arg script_name=$script_name --build-arg test_script_name=$test_script_name --build-arg folder_scripts=$folder_scripts --build-arg module=$module --build-arg image_name=$base_image_name . -t $image_name"
                        }
                        catch (exception){
                            println "The image build is failed"
                            currentBuild.result = pipeline_failure_indicator_string
                            throw exception
                        }

                    }
                }
            }
        }
        stage('Push image to the Dockerhub'){
            when{
                expression { dockerize_string in list_of_choices }
            }
            steps{
                script{
                    docker_hub_repo = 'lidorabo/docker_repo'
                    node(docker_slave_node) {
                        try {
                            withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                                sh "sudo docker login -u=${DOCKER_USERNAME} -p=${DOCKER_PASSWORD}"
                                sh "sudo docker tag $image_name $docker_hub_repo:${image_name.replace(colon,underscore)}"
                                sh "sudo docker push $docker_hub_repo:${image_name.replace(colon,underscore)}"
//
                            }
                          }
                        catch (exception){
                            println "The image upload is failed"
                            currentBuild.result = pipeline_failure_indicator_string
                            throw exception
                        }

                    }
                }
            }
        }
        stage('Running playbook') {
            when{
                expression {  list_of_choices.size() > 0 }
            }
            steps {

                script {
                    print list_of_choices
                    dir(folder_for_run_playbook) {
                        ansiColor('xterm') {
                            ansiblePlaybook(
                                    colorized: true,
                                    credentialsId: '/var/jenkins_home/.ssh/id_rsa',
                                    inventory: 'hosts',
                                    extraVars: [
                                            choices: list_of_choices,
                                            docker_hub_repo: "$docker_hub_repo",
                                            image_name: "${image_name.replace(colon,underscore)}"
                                    ],
                                    extras: '-v'
                                    , playbook: 'main.yaml'

                            )
                        }
                        sh "mv ${hosts_file_name}.bak $hosts_file_name "
                    }

                }
            }
        }
        stage('Push tag to the repository'){
            when{
                expression { dockerize_string in list_of_choices }
            }

            steps{
                script{
                    withCredentials([usernamePassword(credentialsId: 'dcacc4ee-0dab-4a7f-b781-ad2b883bfe68', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        try{
                            sh """
                         git config --global user.name "LidorAbo"
                         git config --global user.email "Lidorabo2@gmail.com"
                         git tag -a $NextVersion -m "Tag for release version of python module"
                         git push  https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/DevOpsINT/Course.git $NextVersion
                            """
                        }
                        catch (exception){
                            println "Pushing tag to git is failed"
                            currentBuild.result = pipeline_failure_indicator_string
                            throw exception
                        }

                    }
                }
            }
        }
        stage('Cleanup') {
            steps {
                deleteDir()
            }

        }
    }
}
