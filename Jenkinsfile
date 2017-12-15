node('master') {
  env.JAVA_HOME = tool 'jdk8'
  env.GRADLE_HOME = tool 'gradle4.2'
  env.GRAILS_HOME = tool 'grails3.2.8'
  env.ANSIBLE_HOME = tool 'ansible2.2.0'
  def mvnHome = tool 'maven3'

  env.PATH = "${env.JENKINS_HOME}/bin:${mvnHome}/bin:${env.GRADLE_HOME}/bin:${env.GRAILS_HOME}/bin:${env.PATH}"
  checkout scm
  if(env.DEPLOY_ENV == "Production") {
    env.DEPLOY_KEY = env.USF_ANSIBLE_VAULT_KEY_PROD
  } else {
    env.DEPLOY_KEY = env.USF_ANSIBLE_VAULT_KEY
  }
  stage('Get Ansible Roles') {
    sh('#!/bin/sh -e\n' + 'rm -rf ansible/roles')
    sh('#!/bin/sh -e\n' + 'ansible-galaxy install -r ansible/requirements.yml -p ansible/roles/ -f')
  }
  stage('Deploy Duo to hosts with ansible') {
    sshagent (credentials: ['jenkins']) {
      sh('#!/bin/sh -e\n' + "ansible-playbook -i ansible/roles/inventory/${env.DEPLOY_ENV.toLowerCase()}/hosts --user=jenkins --vault-password-file=${env.DEPLOY_KEY} ansible/playbook.yml --extra-vars 'target_hosts=${env.DEPLOY_HOST} java_home=${env.JAVA_HOME} deploy_env=${env.DEPLOY_ENV} package_revision=${env.BUILD_NUMBER}' -b -t deploy")
    }
  }
}
