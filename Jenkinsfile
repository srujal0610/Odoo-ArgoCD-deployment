pipeline {
    agent any

    parameters {
        string(name: 'GIT_REPO_URL', defaultValue: 'https://github.com/srujal0610/Odoo-ArgoCD-deployment.git', description: 'GitHub repo with custom addons')
        string(name: 'BRANCH', defaultValue: 'main', description: 'Branch name to pull')
        string(name: 'K8S_NAMESPACE', defaultValue: 'odoo', description: 'Kubernetes namespace')
        string(name: 'POD_LABEL', defaultValue: 'app=odoo', description: 'Pod label selector')
        string(name: 'MOUNT_PATH', defaultValue: '/mnt/extra-addons/custom-addons', description: 'Target path inside Odoo pod')
    }

    environment {
        TMP_TARBALL = 'addons.tar.gz'
    }

    stages {
        stage('Clone Git Repo') {
            steps {
                git url: "${params.GIT_REPO_URL}", branch: "${params.BRANCH}"
            }
        }

        stage('Get Odoo Pod') {
            steps {
                script {
                    env.ODOO_POD = sh(
                        script: "kubectl get pods -n ${params.K8S_NAMESPACE} -l ${params.POD_LABEL} -o jsonpath='{.items[0].metadata.name}'",
                        returnStdout: true
                    ).trim()
                    echo "🟢 Odoo Pod: ${env.ODOO_POD}"
                }
            }
        }

        stage('Copy Addons to Pod') {
            steps {
                sh '''
                echo "📦 Creating tarball..."
                tar czf ${TMP_TARBALL} *

                echo "📤 Copying tarball to pod..."
                kubectl cp ${TMP_TARBALL} ${params.K8S_NAMESPACE}/${ODOO_POD}:/tmp/${TMP_TARBALL}

                echo "📂 Extracting into ${params.MOUNT_PATH}..."
                kubectl exec -n ${params.K8S_NAMESPACE} ${ODOO_POD} -- sh -c "mkdir -p ${params.MOUNT_PATH} && tar xzf /tmp/${TMP_TARBALL} -C ${params.MOUNT_PATH} && rm /tmp/${TMP_TARBALL}"
                '''
            }
        }

        stage('Restart Odoo Pod (Optional)') {
            steps {
                input message: 'Do you want to restart the Odoo pod to apply new addons?'
                sh "kubectl delete pod ${ODOO_POD} -n ${params.K8S_NAMESPACE}"
            }
        }
    }

    post {
        success {
            echo '✅ Custom addons copied and deployed successfully!'
        }
        failure {
            echo '❌ Something went wrong.'
        }
    }
}
