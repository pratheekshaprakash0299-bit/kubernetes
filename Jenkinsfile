pipeline {
  agent any

  environment {
    K8S_API_SERVER = "https://172.31.28.163:6443"
    K8S_NAMESPACE  = "demo"
  }

  stages {

    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Configure kubeconfig') {
      steps {
        withCredentials([string(credentialsId: 'kubernetes-token1', variable: 'KUBE_TOKEN')]) {
          
          sh '''#!/usr/bin/env bash
set -euo pipefail

mkdir -p "$HOME/.kube"

cat > "$HOME/.kube/config" <<EOF
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
    namespace: ${K8S_NAMESPACE}
  name: mycontext
current-context: mycontext
users:
- name: jenkins
  user:
    token: ${KUBE_TOKEN}
EOF

kubectl version --client=true
kubectl cluster-info
'''
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        sh '''#!/usr/bin/env bash
set -euo pipefail

# 1) Create namespace first (safe if already exists)
kubectl apply -f manifests/namespace.yaml

# 2) Quick RBAC check (not required, but helpful)
kubectl auth can-i get pods -n ${K8S_NAMESPACE} || true
kubectl auth can-i apply -f manifests/ -n ${K8S_NAMESPACE} || true

# 3) Apply remaining manifests
kubectl apply -n ${K8S_NAMESPACE} -f manifests/
'''
      }
    }
  }
}
