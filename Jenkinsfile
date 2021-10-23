def remote = [:]
remote.name = "ubuntu"
remote.allowAnyHosts = true
remote.host = "54.93.238.79"
def ID
def IP
node{
    withCredentials([sshUserPrivateKey(credentialsId: '01eb9d49-682c-4e68-94a6-ec77889de9aa', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'userName')]) {
        remote.user = userName
        remote.identityFile = identity
        withCredentials(
        [[
            $class: 'AmazonWebServicesCredentialsBinding',
            accessKeyVariable: 'AWS_ACCESS_KEY_ID',
            credentialsId: '39c07877-ebc4-4f70-a4ca-084feda446e1',  // ID of credentials in kubernetes
            secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
        ]]) {
            stage ("Verifications"){
                sh 'kubectl cluster-info'
                sh 'kubectl get nodes'
                sh 'kubectl version --short'
            }
            stage ("Helm Installation"){
                sh 'curl -LO https://git.io/get_helm.sh'
                sh 'chmod 700 get_helm.sh'
                sh './get_helm.sh'
            }
            stage("Create the Tiller Service Account"){
                sh 'kubectl apply -f https://raw.githubusercontent.com/islajd/kubernetes-prometheus-grafana/master/helm/service-account.yml'
                sh 'kubectl get serviceaccounts -n kube-system'
            }
            stage("Create the service account role binding"){
                sh 'kubectl apply -f https://raw.githubusercontent.com/islajd/kubernetes-prometheus-grafana/master/helm/role-binding.yml'
                sh 'kubectl get clusterrolebindings.rbac.authorization.k8s.io'
            }
            stage("Deploy Tiller"){
                sh 'helm init --service-account tiller --wait'
                sh 'kubectl get pods -n kube-system'
            }
            stage("Create namespace"){
                sh 'kubectl apply -f https://raw.githubusercontent.com/islajd/kubernetes-prometheus-grafana/master/monitoring/namespace.yml'
                sh 'kubectl get namespaces'
            }
            stage ("Deploy Prometheus"){
                sh 'helm repo update'
                sh 'helm install stable/prometheus --namespace monitoring --name prometheus --set server.service.type=LoadBalancer --set server.service.servicePort=8082'
                sh 'kubectl get pods -n monitoring'
            }
            stage("Defining the grafana data sources"){
                sh 'kubectl apply -f https://raw.githubusercontent.com/islajd/kubernetes-prometheus-grafana/master/monitoring/grafana/config.yml'
                sh 'kubectl get configmaps -n monitoring'
            }
            stage("Deploy Grafana"){
                sh 'helm install stable/grafana --namespace monitoring --name grafana --set service.type=LoadBalancer --set service.port=8081 --set persistence.enabled=true'
                sh 'kubectl get pods -n monitoring'
            }
            stage("Get Grafana Password"){
                sh 'kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}"| base64 --decode ; echo'
            }
            stage("describe pipiline results"){
                sh 'kubectl get pods -n monitoring'
                sh 'kubectl get --namespace monitoring services grafana'
                sh 'kubectl describe service prometheus -n monitoring'
                sh 'kubectl describe service grafana -n monitoring'
                // sh 'helm del --purge prometheus' 
                // sh 'helm del --purge grafana' 
                // sh 'kubectl delete namespaces monitoring'
            }
             stage("Slack message")
            {
                slackSend message:"build finished "
            }
        }  
    }
}
