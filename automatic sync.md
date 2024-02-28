###  Automated Sync

* By default, ArgoCD polls Git repositories every 3 minutes to detect changes to the manifests.
* Argo CD can automatically sync apps when it detects differences between the desired manifests in Git, and the live state in the cluster.
    * No need to do manual sync anymore.
    * CI/CD pipelines no longer need direct access.
####  Notes:
* An automated sync will only be performed if the application is OutOfSync.
* Automatic sync will not reattempt a sync if the previous sync attempt against the same commit-SHA and parameters had failed.
* Rollback cannot be performed against an application with automated sync enabled.

<pre>
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata: 
  name: auto-sync-app 
  namespace: argocd
spec: 
  destination: 
    namespace: auto-sync-app
    server: "https://kubernetes.default.svc"
  project: default
  source: 
    path: guestbook-with-sub-directories
    repoURL: "https://github.com/rajeswarithota1715/argocd-private-repo-for-demo.git"
    targetRevision: master
    directory:
      recurse: true
  syncPolicy:
    automated: {}
    syncOptions:
      - CreateNamespace=true
</pre>

###  CLI

<pre>
argocd app create nginx-ingress --repo https://charts.helm.sh/stable --helmchart nginx-ingress --revision 1.24.3 --dest-namespace default --dest-server 
https://kubernetes.default.svc --sync-policy automated
</pre>

####   WEB UI
![Web UI](auto-sync-UI.png)
