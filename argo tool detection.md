###  Tools supported

1.  Helm charts
2.   Kustomize application
3.   Directory of Yaml files
4.   Jsonnet


***Directory of Yaml***

we will use directory of yaml to deploy file in directories and sub directories
<pre>
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata: 
  name: guestbook
  namespace: argocd
spec: 
  destination: 
    namespace: guestbook
    server: "https://kubernetes.default.svc"
  project: default
  source: 
    path: guestbook
    repoURL: "https://github.com/argoproj/argocd-example-apps.git"
    targetRevision: HEAD
    directory:
    recurse: true
</pre>

***Heal Charts***
<pre>
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata: 
    name: guestbook
    namespace: argocd
spec: 
  destination: 
    namespace: guestbook
    server: "https://kubernetes.default.svc"
  project: default
  source: 
    path: helm-guestbook
    repoURL: "https://github.com/argoproj/argocd-example-apps.git"
    targetRevision: HEAD
    helm:
    releaseName: guestboo
</pre>

***kustomize***

we will use kustomize to add prefix ,suffix and labels to resources 
<pre>
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata: 
  name: guestbook
  namespace: argocd
spec: 
  destination: 
    namespace: guestbook
    server: "https://kubernetes.default.svc"
  project: default
  source: 
    path: guestbook-kustomize
    repoURL: "https://github.com/argoproj/argocd-example-apps.git"
    targetRevision: HEAD
    kustomize:
      namePrefix: staging-
      commonLabels:
        app: demo
</pre>
