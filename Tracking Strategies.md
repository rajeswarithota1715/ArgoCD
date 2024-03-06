#  Tracking Strategies

ArgoCD provides several options to track manifests (sources) whether its in Git repos or Helm repos. 

**Git repos tracking**

*  Commit SHA (good for production).
<pre>
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata: 
  name: track-commit 
  namespace: argocd
spec: 
  destination: 
    namespace: track-commit
    server: "https://kubernetes.default.svc"
  project: default
  source: 
    path: guestbook
    repoURL: "https://github.com/rajeswarithota1715/argocd-private-repo-for-demo.git"
    targetRevision: ebf3ff9429ca4bad8edfe1d8c867d816c34352db
    directory:
      recurse: true
  syncPolicy:
    automated: {}
    syncOptions:
      - CreateNamespace=true
</pre>
  
*  Tags (good for production).

<pre>
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata: 
  name: track-git-tag 
  namespace: argocd
spec: 
  destination: 
    namespace: track-git-tag
    server: "https://kubernetes.default.svc"
  project: default
  source: 
    path: guestbook
    repoURL: "https://github.com/rajeswarithota1715/argocd-private-repo-for-demo.git"
    targetRevision: v1
    directory:
      recurse: true
  syncPolicy:
    automated: {}
    syncOptions:
      - CreateNamespace=true
</pre>

* Branch tracking (ex: main branch).
* Symbolic reference (HEAD).


<pre>
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata: 
  name: track-head 
  namespace: argocd
spec: 
  destination: 
    namespace: track-head
    server: "https://kubernetes.default.svc"
  project: default
  source: 
    path: guestbook
    repoURL: "https://github.com/rajeswarithota1715/argocd-private-repo-for-demo.git"
    targetRevision: HEAD
    directory:
      recurse: true
  syncPolicy:
    automated: {}
    syncOptions:
      - CreateNamespace=true
  </pre>

  ![git](https://github.com/rajeswarithota1715/ArgoCD/blob/ecb7be50055449556ddb2574eee891e6117011e2/tracking%20strategies.PNG)

**Helm repos tracking** 

Helm always use semantic versioning

* Specific version v1.2
*  Range 1.2.* or >=1.2.0 <1.3.0.


  <pre>
  apiVersion: argoproj.io/v1alpha1
kind: Application
metadata: 
  name: track-helm-range 
  namespace: argocd
spec: 
  destination: 
    namespace: track-helm-range
    server: "https://kubernetes.default.svc"
  project: default
  source: 
    chart: sealed-secrets
    repoURL: "https://charts.bitnami.com/bitnami"
    targetRevision: 1.*
  syncPolicy:
    automated: {}
    syncOptions:
      - CreateNamespace=true
</pre>
*   Latest * or >=0.0.0

<pre>
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata: 
  name: track-helm-latest-version 
  namespace: argocd
spec: 
  destination: 
    namespace: track-helm-latest-version
    server: "https://kubernetes.default.svc"
  project: default
  source: 
    chart: ingress-nginx
    repoURL: "https://kubernetes.github.io/ingress-nginx"
    targetRevision: '*'
  syncPolicy:
    automated: {}
    syncOptions:
      - CreateNamespace=true
</pre>

![helm](https://github.com/rajeswarithota1715/ArgoCD/blob/ecb7be50055449556ddb2574eee891e6117011e2/tracking%20strategies1.PNG)
