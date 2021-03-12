pipeline {
    agent {
        kubernetes {
            // Rather than inline YAML, in a multibranch Pipeline you could use: yamlFile 'jenkins-pod.yaml'
            // Or, to avoid YAML:
            // containerTemplate {
            //     name 'shell'
            //     image 'ubuntu'
            //     command 'sleep'
            //     args 'infinity'
            // }
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: shell
    image: ubuntu
    command:
    - sleep
    args:
    - infinity
  - name: helmcontainer
    image: projectwave/helm:awscliv2
    command:
    - sleep
    args:
    - infinity
'''
            // Can also wrap individual steps:
            // container('shell') {
            //     sh 'hostname'
            // }
            defaultContainer 'shell'
        }
    }
    environment{
        PUBLIC_KEY=credentials('gpg_public_key')
        PRIVATE_KEY=credentials('gpg_private_key')
        GPG_PASSPHRASE=credentials('gpg_passphrase')
        KUBECONFIG  = credentials('kubernetes-central')
    }
    stages {
        stage('Main') {
            steps {
                sh 'hostname'
            }
        }
        stage('helm') {
            steps {
                container('helmcontainer'){
                        sh 'helm version'
                        sh 'gpg --version'
                        sh 'export tty=/dev/ttys000'
                        sh 'export GPG_TTY=/dev/ttys000'
                        sh 'gpg --import ${PUBLIC_KEY}'
                        sh 'gpg --batch --pinentry-mode loopback --passphrase ${GPG_PASSPHRASE} --allow-secret-key-import --import ${PRIVATE_KEY}'
                        sh 'gpg --list-keys'
                        sh 'echo $HOME'
                        sh 'export GPG_OPTS="--pinentry-mode loopback"'
                        sh 'echo ${GPG_OPTS}'
                        sh 'gpg --batch --passphrase ${GPG_PASSPHRASE} --allow-secret-key-import --import ${PRIVATE_KEY} >~/.gnupg/secring.gpg'
                        sh 'helm plugin install https://github.com/jkroepke/helm-secrets --version v3.5.0'
                        sh ' helm secrets dec postgresqlhelmsecret/secrets.yaml'
                }
            }
        }
    }
}

