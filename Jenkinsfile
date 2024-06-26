#!/usr/bin/groovy

@Library(['github.com/indigo-dc/jenkins-pipeline-library@1.2.3']) _

def docker_push(id_this) {
    println("[DEBUG] ${docker_registry}")
    println("[DEBUG] ${docker_repository}")
    docker.withRegistry(docker_registry, 
        docker_registry_credentials) {
        id_this.push()
    }
}

// define which TensorFlow versions to use
def getTFVers(){
    return ["2.9.3", "2.10.0", "2.11.0", "2.12.0", "2.13.0"]
}

// define which pytorch tags to use
def getPyTorchTags(){
    return ["1.11.0-cuda11.3-cudnn8-runtime", "1.12.0-cuda11.3-cudnn8-runtime", "1.13.0-cuda11.6-cudnn8-runtime", "2.0.0-cuda11.7-cudnn8-runtime", "2.1.0-cuda11.8-cudnn8-runtime"]
}

// define which pytorch versions to use
def getPyTorchVers(){
    return ["1.11", "1.12", "1.13", "2.0", "2.1"]
}

pipeline {
    agent {
        label 'docker-build'
    }

    environment {
        docker_image_name = "ai4os-dev-env"
    }

    stages {

        stage("Variable initialization") {
            environment {
                AI4OS_REGISTRY_CREDENTIALS = credentials('AIOS-registry-credentials')
            }
            steps {
                checkout scm
                script {
                    withFolderProperties{
                        docker_registry = env.AI4OS_REGISTRY
                        docker_registry_credentials = env.AI4OS_REGISTRY_CREDENTIALS
                        docker_repository = env.AI4OS_REGISTRY_REPOSITORY
                    }
                    // get docker image name from metadata.json
                    meta = readJSON file: "metadata.json"
                    image_name = meta["sources"]["docker_registry_repo"].split("/")[1]
                    println("[DEBUG] ${docker_registry}")
                    println("[DEBUG] ${docker_repository}")
                }
            }
        }

        stage('Docker image building (ubuntu)') {
            when {
                anyOf {
                   branch 'master'
                   buildingTag()
               }
            }
            steps{
                checkout scm
                script {
                    // build different tags
                    id = "${env.docker_image_name}"

                    // Finally, we put all AI4OS components in 
                    // ubuntu 20.04 image without deep learning framework
                    image = docker_repository + "/" + image_name + ":" + "u20.04"
                    println("[DEBUG] ${image}")
                    base_image="ubuntu"
                    base_tag="20.04"
                    id_u2004 = docker.build(image,
                                            "--no-cache --force-rm --build-arg image=${base_image} --build-arg tag=${base_tag} .")
                    docker.withRegistry(docker_registry, 
                        docker_registry_credentials) {
                        id_u2004.push()
                    }
                    println("[DEBUG] NOW BUILDING VIA FUNCTION")
                    docker_push(id_u2004)

                    id_u2204 = DockerBuild(image,
                                           tag: ['u22.04'],
                                           build_args: ["image=ubuntu",
                                                        "tag=22.04"])
                    docker_push(id_u2204[0]) // DockerBuild returns array (!)

                    // immediately remove local image
                    id_this = id_u2004[0]
                    sh("docker rmi --force \$(docker images -q ${id_this})")
                    id_this = id_u2204[0]
                    sh("docker rmi --force \$(docker images -q ${id_this})")
                    //sh("docker rmi --force \$(docker images -q ubuntu:18.04)")
                }
            }
            post {
                failure {
                    DockerClean()
                }
            }
        }
    
        stage('Docker image building (pytorch)') {
            when {
                anyOf {
                   branch 'master'
                   buildingTag()
               }
            }
            steps{
                checkout scm
                script {
                    // build different tags
                    id = "${env.dockerhub_repo}"

                    // pyTorch
                    pytorch_tags = getPyTorchTags()
                    pytorch_vers = getPyTorchVers()
                    p_vers = pytorch_vers.size()

                    // CAREFUL! For-loop might fail in some Jenkins versions
                    // Other options: 
                    // https://stackoverflow.com/questions/37594635/why-an-each-loop-in-a-jenkinsfile-stops-at-first-iteration
                    for(int j=0; j < p_vers; j++) {
                        tag_id = ['pytorch'+pytorch_vers[j]]
                        pytorch_tag = pytorch_tags[j]
                        id_pytorch = DockerBuild(id,
                                                 tag: tag_id,
                                                 build_args: ["image=pytorch/pytorch",
                                                              "tag=${pytorch_tag}"])

                        docker.withRegistry(env.AI4OS_REGISTRY, 
                                            credentials('AIOS-registry-credentials')) {
                            docker.image(id_pytorch).push()
                        }

                        // immediately remove local image
                        id_this = id_pytorch[0]
                        sh("docker rmi --force \$(docker images -q ${id_this})")
                    }
               }
            }
            post {
                failure {
                   DockerClean()
                }
            }
        }

        stage('Docker image building (tensorflow)') {
            when {
                anyOf {
                   branch 'master'
                   buildingTag()
               }
            }
            steps{
                checkout scm
                script {
                    // build different tags
                    id = "${env.dockerhub_repo}"

                    // TensorFlow
                    tf_vers = getTFVers()
                    n_vers = tf_vers.size()

                    // CAREFUL! For-loop might fail in some Jenkins versions
                    // Other options: 
                    // https://stackoverflow.com/questions/37594635/why-an-each-loop-in-a-jenkinsfile-stops-at-first-iteration
                    for(int j=0; j < n_vers; j++) {
                        tags = ['tf'+tf_vers[j]+'-cpu', 
                                'tf'+tf_vers[j]+'-gpu'] 

                        tf_tags = [tf_vers[j],
                                   tf_vers[j]+'-gpu']

                        for(int i=0; i < tags.size(); i++) {
                            tag_id = [tags[i]]
                            // tag last cpu tag as "latest"
                            if (j == (n_vers - 1) && tags[i].contains("-cpu")) {
                                tag_id = [tags[i], 'latest']
                            }
                            // tag last gpu tag as "latest-gpu"
                            if (j == (n_vers - 1) && tags[i].contains("-gpu")) {
                                tag_id = [tags[i], 'latest-gpu']
                            }
                            tf_tag = tf_tags[i]
                            id_docker = DockerBuild(id,
                                                    tag: tag_id,
                                                    build_args: ["image=tensorflow/tensorflow",
                                                                 "tag=${tf_tag}"])

                            docker.withRegistry(env.AI4OS_REGISTRY,
                                                credentials('AIOS-registry-credentials')) {
                                docker.image(id_docker).push()
                            }

                            // immediately remove local image
                            id_this = id_docker[0]
                            sh("docker rmi --force \$(docker images -q ${id_this})")
                            //sh("docker rmi --force \$(docker images -q tensorflow/tensorflow:${tf_tag})")
                        }
                    }
               }
            }
            post {
                failure {
                   DockerClean()
                }
            }
        }

    }
}
