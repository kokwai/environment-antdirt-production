pipeline {
  options {
    disableConcurrentBuilds()
  }
  agent {
    label "jenkins-maven"
  }
  environment {
    DEPLOY_NAMESPACE = "jx-production"
  }
  stages {
    stage('Validate Environment') {
      steps {
        container('maven') {
          dir('env') {
            sh '''
                curl -kLo helm-linux-amd64-v2.9.1.tar.gz https://9.30.118.226:8443/api/cli/helm-linux-amd64.tar.gz
                tar -xvf helm-linux-amd64-v2.9.1.tar.gz
                rm helm-linux-amd64-v2.9.1.tar.gz
                cp -f linux-amd64/helm /usr/bin/helm
                rm -rf linux-amd64
                wget https://gist.githubusercontent.com/kokwai/fe352638f8f2495176e337a9c7a1efa3/raw/103e34e0a66fe3cc53b8738c67a4b9896ce9cae3/helm_wrapper.sh
                chmod 755 helm_wrapper.sh
                which helm
                helm init --client-only --skip-refresh
                curl -kLo cloudctl https://9.30.118.226:8443/api/cli/cloudctl-linux-amd64
                chmod 755 cloudctl
                ./cloudctl login --skip-ssl-validation -a https://9.30.118.226:8443 -u admin -p admin -c id-mycluster-account -n default
                cp /home/jenkins/.cloudctl/clusters/mycluster/cert.pem /home/jenkins/.helm/cert.pem
                cp /home/jenkins/.cloudctl/clusters/mycluster/key.pem /home/jenkins/.helm/key.pem
                rm cloudctl
                ls -R
                helm version --tls
                ./helm_wrapper.sh version
                jx edit helmbin helm_wrapper.sh
            '''
            sh 'jx step helm build'
          }
        }
      }
    }
    stage('Update Environment') {
      when {
        branch 'master'
      }
      steps {
        container('maven') {
          dir('env') {
            sh 'jx step helm apply'
          }
        }
      }
    }
  }
}
