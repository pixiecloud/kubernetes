.
├── README.md
├── VERSION
├── argo-artifacts
│   ├── application.yaml
│   ├── kustomization.yaml
│   ├── recon-pipeline.yaml
│   ├── source-pipeline.yaml
│   └── world-pipeline.yaml
├── docker
│   └── worker
│       ├── Dockerfile
│       └── runner-scripts
│           └── start.sh
├── k8s
│   ├── configmap.yaml
│   ├── deployment.yaml
│   ├── secret.yaml
│   └── service.yaml
└── scripts
    └── bump_version.sh

## building with the pipeline
run the pipeline to build and push the github-runner docker image 

## After building, create an AWS Secret and GH PAT secret
- If you pushed your image to ecr, you will need to create an ecr aws secret

```sh
kubectl create secret docker-registry ecr-secret \
  --docker-server=368085106192.dkr.ecr.us-east-1.amazonaws.com \
  --docker-username=AWS \
  --docker-password=$(aws ecr get-login-password --region us-east-1) \
  #--namespace=argo \
  --docker-email=example@example.com  \
  --dry-run=client -o yaml > awsecr-secret.yaml
```

```sh
kubectl create secret generic github-token-secret --from-literal=GH_TOKEN=github_pat_11A34YAFQ0U4xr.........
```

```sh
echo -n 'your-token-here' | base64

kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/secret.yaml
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl get pods -l app=github-actions-runner

```

## OPTION 2: building and running with docker locally
```sh
docker build -t cafanwii/github-actions-runner .
```

```sh
docker run -d --name github-actions-runner \
  -e GH_OWNER=sosotechnologies \
  -e GH_REPOSITORY=runners \
  -e GH_TOKEN=github_pat_11A34YAF... \   # your GH PAT
  cafanwii/github-actions-runner 
```

```sh
docker logs github-actions-runner
```





## Extras ###################

### Aws Ecr Login
```sh
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <your-account-number>.dkr.ecr.us-east-1.amazonaws.com

docker pull 368085106192.dkr.ecr.us-east-1.amazonaws.com/runners:latest

docker run -itd <your-account-number>.dkr.ecr.us-east-1.amazonaws.com/runners:latest

```

### Force  git push
```sh 
git checkout --orphan temp_branch
git add -A
git commit -am "Initial commit"
git branch -D main
git branch -m main
git push -f origin main
```

<!-- git checkout --orphan temp_branch: Creates a new branch with no commit history.
git add -A: Adds all files to the staging area.
git commit -am "Initial commit": Commits all files with the message "Initial commit."
git branch -D main: Deletes the old main branch.
git branch -m main: Renames the current branch to main.
git push -f origin main: Force pushes the new main branch to the remote repository, overwriting the existing history. -->





