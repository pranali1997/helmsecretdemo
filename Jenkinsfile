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
                    sh 'echo $(tty)'
                    sh 'GPG_TTY=/dev/ttys002'
                    sh 'export GPG_TTY'
                    sh 'gpg --import public.key'
                    sh 'gpg --batch --passphrase ${GPG_PASSPHRASE} --allow-secret-key-import --import private.key'
                    sh 'gpg --list-keys'
                    sh 'helm plugin install https://github.com/jkroepke/helm-secrets --version v3.5.0'
                    sh ' helm secrets dec postgresqlhelmsecret/secrets.yaml'
                    //sh 'helm secrets install postgresqldemo postgresqlhelmsecret --values postgresqlhelmsecret/secrets.yaml -n infra --kubeconfig=${KUBECONFIG}'
                }
            }
        }
    }
}

