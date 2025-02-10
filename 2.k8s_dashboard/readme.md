helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/

helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard


kaf sa.yml

kubectl get secret admin-user -n kubernetes-dashboard -o jsonpath="{.data.token}" | base64 -d

kaf crb.yml

<!-- Note: Convert below cluster ip service into nodePort service and try  accessing from browser -->

k edit svc kubernetes-dashboard-kong-proxy

###  Hit below commadn & get the  token & access the dashboard

kubectl get secret admin-user -n kubernetes-dashboard -o jsonpath="{.data.token}" | base64 -d




