#!/usr/bin/groovy

@Library(['github.com/indigo-dc/jenkins-pipeline-library@1.2.3']) _

// define which TensorFlow versions to use
def getTFVers(){
    return ["1.12.0", "1.14.0"]
}

pipeline {
    agent {
        label 'docker-build'
    }

    environment {
        dockerhub_repo = "vykozlov/deep-oc-generic-dev"
    }

    stages {

        stage('Validate metadata') {
            steps {
                checkout scm
                sh 'deep-app-schema-validator metadata.json'
            }
        }

        stage('Docker image building') {
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

                    tf_vers = getTFVers()

                    for(int j=0; j < tf_vers.size(); j++) {
                        tags = ['tf'+tf_vers[j]+'-cpu', 
                                'tf'+tf_vers[j]+'-gpu'] 

                        tf_tags = [tf_vers[j]+'-py3',
                                   tf_vers[j]+'-gpu-py3']

                        for(int i=0; i < tags.size(); i++) {
                           tf_tag = tf_tags[i]
                           id_docker = DockerBuild(id,
                                             tag: [tags[i]],
                                             build_args: ["tag=${tf_tag}",
                                                          "pyVer=python3"])
                           DockerPush(id_docker)
                           DockerClean()
                        }
                    }

                    tags_1120 = ['tf1.12.0-cpu-py2', 
                                 'tf1.12.0-gpu-py2',
                                 'tf1.12.0-cpu-py36',
                                 'tf1.12.0-gpu-py36' ]

                    tf_images_1120 = ['tensorflow/tensorflow',
                                      'tensorflow/tensorflow',
                                      'deephdc/tensorflow',
                                      'deephdc/tensorflow']

                    tf_tags_1120 = ['1.12.0', '1.12.0-gpu', 
                                    'tf-py36', 'tf-gpu-py36']

                    pyVes_1120 = ['python', 'python', 'python3', 'python3']

                    // CPU + python3 (aka default)
                    //id_cpu = DockerBuild(id,
                    //                     tag: ['latest','tf-cpu'],
                    //                     build_args: ["tag=${env.tf_ver}-py3",
                    //                                  "pyVer=python3"])

                    // GPU + python3
                    //id_gpu = DockerBuild(id,
                    //                     tag: ['tf-gpu'], 
                    //                     build_args: ["tag=${env.tf_ver}-gpu-py3",
                    //                                  "pyVer=python3"])

                    // CPU + python2
                    //id_cpu_py2 = DockerBuild(id,
                    //                        tag: ['tf-py2'], 
                    //                        build_args: ["tag=${env.tf_ver}",
                    //                                     "pyVer=python"])

                    // GPU + python2
                    //id_gpu_py2 = DockerBuild(id,
                    //                         tag: ['tf-gpu-py2'], 
                    //                         build_args: ["tag=${env.tf_ver}-gpu",
                    //                                      "pyVer=python"])

                    // CPU + python3.6 (ubuntu18.04 + python3.6 + tf1.12.0)
                    //id_cpu_py36 = DockerBuild(id,
                    //                          tag: ['tf-py36'],
                    //                          build_args: ["image=deephdc/tensorflow",
                    //                                       "tag=1.12.0-py36",
                    //                                       "pyVer=python3"])

                    // GPU + python3.6 (ubuntu18.04 + python3.6 + tf1.12.0)
                    //id_gpu_py36 = DockerBuild(id,
                    //                          tag: ['tf-gpu-py36'],
                    //                          build_args: ["image=deephdc/tensorflow",
                    //                                       "tag=1.12.0-gpu-py36",
                    //                                       "pyVer=python3"])
                    //// ubuntu-only (18.04 + python3.6)
                    //id_u1804 = DockerBuild(id,
                    //                       tag: ['u18.04'],
                    //                       build_args: ["image=ubuntu",
                    //                                    "tag=18.04",
                    //                                    "pyVer=python3"])
                }
            }
            post {
                failure {
                    DockerClean()
                }
            }
        }

        //stage('Docker Hub delivery') {
        //
        //    steps{
        //        script {
        //            //DockerPush(id_cpu)
        //            //DockerPush(id_gpu)
        //            //DockerPush(id_cpu_py2)
        //            //DockerPush(id_gpu_py2)
        //            //DockerPush(id_cpu_py36)
        //            //DockerPush(id_gpu_py36)
        //            //DockerPush(id_u1804)
        //        }
        //    }
        //    post {
        //        failure {
        //            DockerClean()
        //        }
        //        always {
        //            cleanWs()
        //        }
        //    }
        //}

        stage("Render metadata on the marketplace") {
            when {
                allOf {
                    branch 'master'
                    changeset 'metadata.json'
                }
            }
            steps {
                script {
                    //def job_result = JenkinsBuildJob("Pipeline-as-code/deephdc.github.io/pelican")
                    job_result_url = job_result.absoluteUrl
                }
            }
        }
    }
}
