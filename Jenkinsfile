// Uses Declarative syntax to run commands inside a container.
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
    image: projectwave/helm
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
        PUBLIC_KEY=credentials('gpg-ownertrust')
        PRIVATE_KEY=credentials('gpg-secret')
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
                    sh 'export GPG_TTY=$(tty)'
                    sh 'gpg --batch --import $gpg_secret'
                    sh 'gpg --import-ownertrust $gpg_trust'
                    sh 'gpg --list-keys'
                    sh 'helm plugin install https://github.com/jkroepke/helm-secrets --version v3.5.0'
                }
            }
        }
        stage (check password) {
            steps {
                container('helmcontainer'){
                sh '''
                    cd helm-secret-demo
                    git secret reveal -p '$gpg_passphrase'
                    git secret cat bankservice/.env
                '''
                }
            }
        }
    }
}
