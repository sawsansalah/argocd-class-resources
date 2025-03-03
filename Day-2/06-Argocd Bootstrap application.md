1. Create a new YAML file called `argocd.yaml` and add the following:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: argocd
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://gitlab.com/[your username]/samplegitopsapp.git'
    path: argo-cd
    targetRevision: main
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: argocd
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
```

3. Apply the above configuration by running:

   ```bash
   kubectl apply -f argocd.yaml
   ```

4. Go to the UI of Argo CD.

5. Login using the username of `admin` and your password

6. Delete the `nginx` application from the UI. You need to type the name of the application `nginx` in the dialog box for the deletion operation to complete.

7. Go to the terminal and in the `argocd` directory create a new application manifest for HTTPbin. The filename should be `httpbin.yaml` and the contents should be as follows:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: httpbin
  namespace: argocd
spec:
  project: default
  source:
    chart: httpbin
    repoURL: https://matheusfm.dev/charts
    targetRevision: 0.1.1
    helm:
      releaseName: httpbin
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: default
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
```

8. Create and push a merge request:

```bash
git checkout -b feature/add-httpbin-chart
git add httpbin.yaml
git commit -m "Adds the HTTPbin chart"
git push --set-upstream origin feature/add-httpbin-chart
```

9. Copy the MR link that is displayed and paste it in new browser tab.

10. Approve and merge the MR.

11. Go to ArgoCD UI and click Refresh on the argocd application. You should see the HTTPbin application appear and the icon reflecting that it is a Helm chart.

12. Go back to the terminal and open the `httpbin.yaml`.

13. The contents of the file should look as follows:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: httpbin
  namespace: argocd
spec:
  project: default
  source:
    chart: httpbin
    repoURL: https://matheusfm.dev/charts
    targetRevision: 0.1.1
    helm:
      releaseName: httpbin
      values: |
        service:
          type: NodePort
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: default
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
```

14. Save the file.

15. Create a merge request by running the following commands:

```bash
git checkout -b feature/httpbin-service-type
git add httpbin.yaml
git checkout -m "Changes the HTTPbin service type to NodePort"
git push --set-upstream origin feature/httpbin-service-type
```

16. Copy the link from the output and paste it in a new browser tab. Create, approve, and merge the MR.

17. Move to the Argo CD dashboard and refresh the argocd application.

18. Move back to the terminal and check the service type again by running:

    ```bash
    kubectl get svc
    ```

