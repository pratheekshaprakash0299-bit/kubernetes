pipeline {
    agent any

    environment {
        K8S_API_SERVER = "https://172.31.11.205:6443"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([string(credentialsId: 'token-k8s', variable: 'KUBE_TOKEN')]) {
                    sh '''
                        mkdir -p $HOME/.kube

                        cat <<EOF > $HOME/.kube/config
apiVersion: v1
kind: Config
clusters:
- cluster:
    server: ${K8S_API_SERVER}
    insecure-skip-tls-verify: true
  name: mycluster
contexts:
- context:
    cluster: mycluster
    user: jenkins
    namespace: demo
  name: mycontext
current-context: mycontext
users:
- name: jenkins
  user:
    token: ${KUBE_TOKEN}
EOF

                        kubectl apply -f manifests/
                    '''
                }
            }
        }
    }
}
