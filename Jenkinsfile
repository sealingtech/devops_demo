#!/usr/bin/groovy

// load pipeline functions
// Requires pipeline-github-lib plugin to load library from github

@Library('github.com/lachie83/jenkins-pipeline@dev')
def pipeline = new io.estrado.Pipeline()


node {
  def app



  def pwd = pwd()
  def chart_dir = "$pwd/helm/"
  def tool_name = "devopsdemo"
  def container_dir = "$pwd/container/"
  def custom_image = "images.devopsdemo"
  def user_id = ''
  wrap([$class: 'BuildUser']) {
      echo "userId=${BUILD_USER_ID},fullName=${BUILD_USER},email=${BUILD_USER_EMAIL}"
      user_id = "${BUILD_USER_ID}"
  }

  sh "env"

  def container_tag = "gcr.io/edcop-dev/$user_id-$tool_name"

  stage('Clone repository') {
      /* Let's make sure we have the repository cloned to our workspace */
      checkout scm
  }

  stage('Build image') {
      /* This builds the actual image; synonymous to
       * docker build on the command line */
      println("Building $container_tag:$env.BUILD_ID")

      app = docker.build("$container_tag:$env.BUILD_ID","./container/")
  }


  stage('Push image') {
      /* Finally, we'll push the image with two tags:
       * First, the incremental build number from Jenkins
       * Second, the 'latest' tag.
       * Pushing multiple tags is cheap, as all the layers are reused. */
      docker.withRegistry('https://gcr.io/edcop-dev/', 'gcr:edcop-dev') {
          app.push("$env.BUILD_ID")
      }
  }

  stage('Getting Vulnerability scan data') {
    sh "if ! gcloud beta container images list-tags  gcr.io/edcop-dev/$user_id-$tool_name --format='value(TAGS,VULNERABILITIES)' --filter='tags=$env.BUILD_ID' | awk '{print $2}' | grep -q CRITICAL; then exit 1;fi"
  }

  stage('helm lint') {
      sh "helm lint $tool_name"
  }

  stage('helm deploy' ) {
      sh "helm install --set $custom_image='$container_tag:$env.BUILD_ID' --name='$user_id-$tool_name-$env.BUILD_ID' $tool_name"
  }

  stage('sleeping 20 seconds') {
    sleep(20)
  }

  stage('Verifying running pods') {
    def number_ready=sh(returnStdout: true, script: "kubectl get deploy demo-devops-deployment  -o jsonpath={.status.numberReady}").trim()
    def number_scheduled=sh(returnStdout: true, script: "kubectl get deploy demo-devops-deployment  -o jsonpath={.status.currentNumberScheduled}").trim()

    println("Ready pods: $number_ready  Scheduled pods: $number_scheduled")

    if(number_ready==number_scheduled) {
      println("Pods are running")
    } else {
      println("Some or all Pods failed")
      error("Some or all Pods failed")
    }
  }

  stage('Verifying 200 code') {
    println("make sure 200 code is returned")
    sh "curl -f apps.devops.stech.demo"
  }
}
