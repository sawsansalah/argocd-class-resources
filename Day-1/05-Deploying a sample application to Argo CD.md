* This lecture assumes that you are using GitLab. For other Git services, please consult the provider's documentation.
* Go to gitlab.com and login.
* Create a new project called SampleGitOpsApp. The project should be blank, private, and without a README file.* Go back to the terminal and run the following commands:
```bash
cd myapp
git remote add origin git@gitlab.com:[your username]/samplegitopsapp.git
git push -u origin --all
```
* Go back to Gitlab and refresh the page.
* Click on the profile picture and select preferences. Select peronsl access tokens.
* Create one called argo cd. The access level should be api .
* Copy the value of the token.
* On the terminal, create a secret by running the following command:
```bash
argocd repo add "https://gitlab.com/devops9663651/my-sample-argocd.git"  --username "
[your username(full email)]" --password "[your personal token]"
```
* Create the Argo CD directory by running the following commands:
```bash
mkdir argo-cd
cd argo-cd
```
* Create the Nginx installation file as follows:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://gitlab.com/devops9663651/my-sample-argocd.git
    path: manifests
    targetRevision: main
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      selfHeal: true
      prune: true

```
Apply the manifest to the cluster as follows:
```bash
kubectl apply -f nginx.yaml
```
Go to the UI and view the applications. Make sure that the sync status is OK. Go back to the terminal and view the running pods and services:
```bash
kubectl get pods
kubectl get services
```
Create a Git workflow as follows:
```bash
git checkout -b feature/increase-replicas
# modify the deployment replicas to be three
git add base/deployment.yaml
git commit -m "Inreases the replica count"
git push --set-upstream origin feature/increase-replicas
```
Go to Gitlab by copying the link from the output message of the above command. Approve and merge the MR.
Go back to the terminal and run the following:
```bash
argocd app sync nginx
kubectl get pods
```