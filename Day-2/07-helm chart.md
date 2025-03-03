

22. Create a new Helm chart in the root directory by running the following command:

    ```bash
    helm create httpd
    ```

23. Go inside the directory and change the values file as follows:

```yaml
# values.yaml
image:
  repository: httpd
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: latest
```

24. Check the version of the chart in the Chart.yaml file.

25. Package the chart by running:

    ```bash
    helm package .
    ```

26. Check the file that was created by running:

    ```bash
    ls -ltr
    ```

27. Get the project ID from Gitlab by going to the project page and clicking on settings. Copy the ID.

28. In the terminal, upload the package by running the following command:

    ```bash
    curl --request POST --form 'chart=@httpd-0.1.0.tgz' --user "[your username]:[your token]" https://gitlab.com/api/v4/projects/[your project id]/packages/helm/api/stable/charts
    ```

29. Create the repository credentials using the following command:

    ```bash
    argocd repocreds add https://gitlab.com/api/v4/projects/[your project id]/packages/helm/stable --username [your username] --password [your personal token]
    ```

30. Create a new branch to add the busybox manifest:

    ```bash
    git checkout -b feature/httpd
    ```

31. Create a new application in the argocd directory called `busybox.yaml` and add the following content:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: httpd
  namespace: argocd
spec:
  project: default
  source:
    chart: httpd
    repoURL: https://gitlab.com/api/v4/projects/[project id]/packages/helm/stable
    targetRevision: 0.1.0
    helm:
      releaseName: httpd
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: default
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
```

32. Push the changes to Gitlab

    ```bash
    git add -A
    git commit -m "Adds the Apache Argo CD manifest"
    git push --set-upstream origin feature/httpd
    ```

33. Copy the link that was generated in the output, paste in a new browser tab. Approve and merge the MR to the main branch.

34. Move to the Argo CD UI and click on the Refresh button on the Argo CD applciation to sync the changes.

35. Move to the terminal and view the BusyBox pods by running:

    ```bash
    kubectl get pods
    ```

36. Create a new branch to change Nginx installation method from manifests to the Helm directory:

    ```bash
    git checkout -b feature/nginx-helm
    ```

37. Change the `nginx.yaml` manifest to look as follows:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://gitlab.com/[your username]/samplegitopsapp.git'
    path: mychart # changed
    targetRevision: main
    # Add this
    helm:
      releaseName: nginx
      values: |
        replicaCount: 2
    # till here
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: default
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
```

38. Push the change to Gitlab:

    ```bash
    git add -A
    git commit -m "Changes the Nginx installation from plain manifests to Helm"
    git push --set-upstream origin feature/nginx-helm
    ```

39. Copy the link that was generated in the output and paste it in a new browser tab. Approve and merge the MR.

40. Go to the Argo CD UI and click Refresh on the argocd application. Make sure that the nginx application has changed the path to `mychart`.

41. Go to the terminal and show the new pods by running:

    ```bash
    kubectl get pods
    ```
