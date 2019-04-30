pipeline {

    options {
        timestamps()
        timeout(time: 45, unit: 'MINUTES', activity: true)
    }

    agent {
        node {
            label "cloud-ccp-ci"
        }
    }
    environment {
        /* use lowercase SOCOK8S_ENVNAME. CaaSP Velum doesn't like it otherwise */
        SOCOK8S_ENVNAME = "cloud-socok8s-${env.BRANCH_NAME.toLowerCase()}-${env.BUILD_NUMBER}"
        OS_CLOUD = "engcloud-cloud-ci"
        KEYNAME = "engcloud-cloud-ci"
        DELETE_ANYWAY = "YES"
        SOCOK8S_DEVELOPER_MODE = "True"
        DEPLOYMENT_MECHANISM = "openstack"
        ANSIBLE_STDOUT_CALLBACK = "yaml"
    }

    stages {
        stage('Show environment information') {
            steps {
                sh 'printenv'
            }
        }

        stage('Setup network') {
            steps {
                sh "./run.sh deploy_network"
            }
        }
        stage('Setup hosts') {
            parallel {
                stage('Setup CaaSP') {
                    steps {
                        sh "./run.sh deploy_caasp"
                    }
                }
                stage('Setup SES') {
                    steps {
                        sh "./run.sh deploy_ses"
                    }
                }
                stage('Setup Deployer') {
                    steps {
                        sh "./run.sh deploy_ccp_deployer"
                    }
                }
            }
        }
        stage('Enroll CaaSP workers') {
            steps {
                sh "./run.sh enroll_caasp_workers"
            }
        }
        stage('Setup CaaSP workers and apply upstream patches') {
            parallel {
                stage('Setup CaaSP workes for OpenStack Helm') {
                    steps {
                        sh "./run.sh setup_caasp_workers_for_openstack"
                    }
                }
                stage('Apply upstream patches') {
                    steps {
                        sh "./run.sh patch_upstream"
                    }
                }
            }
        }
        stage('Deploy OpenStack Helm') {
            steps {
                sh "./run.sh build_images"
                sh "./run.sh deploy_osh"
            }
        }
    }

    post {
        failure {
            script {
                if (env.hold_instance_for_debug == 'true') {
                    echo "You can reach this node by connecting to its floating IP as root user, with the default password of your image."
                    timeout(time: 3, unit: 'HOURS') {
                        input(message: "Waiting for input before deleting  env ${SOCOK8S_ENVNAME}.")
                    }
                }
            }
        }
        always {
            script {
                sh './run.sh teardown'
            }
        }
    }
}
