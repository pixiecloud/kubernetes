## create the PAT secret
```sh
kubectl create secret generic github-pat-secret \
  -n github-runner-system \
  --from-literal=GH_TOKEN='<YOUR_PAT_HERE>'
```

## then create a deployment with your repo name
```sh
echo -n 'A34YAFQILD7IDFMINGVF6Z3H4KHPM' | base64   ## echo this runner token and add to secret.yaml

kubectl apply -f configmap.yaml
kubectl apply -f secret.yaml             ## this is the repo runner secret
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl -n github-runner-system get pods -l app=github-actions-runner
```