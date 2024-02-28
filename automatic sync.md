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

###   Automated Pruning

*   Default – no prune: when automated sync is enabled, by default for safety automated sync will not delete resources when Argo CD detects the resource is no longer defined in Git.
*   Pruning can be enabled to delete resources automatically as part of the automated sync

### CLI

<pre>
argocd app create nginx-ingress --repo https://charts.helm.sh/stable --helmchart nginx-ingress --revision 1.24.3 --dest-namespace default --dest-server 
https://kubernetes.default.svc --auto-prune
</pre>

### without enabling auro prune
<pre>
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata: 
  name: auto-pruning-demo 
  namespace: argocd
spec: 
  destination: 
    namespace: auto-pruning-demo
    server: "https://kubernetes.default.svc"
  project: default
  source: 
    path: guestbook-with-sub-directories
    repoURL: "https://github.com/rajeswarithota1715/argocd-private-repo-for-demo.git"
    targetRevision: master
    directory:
      recurse: true
  syncPolicy:
    automated:   {}
    syncOptions:
      - CreateNamespace=true
</pre>

 Apply the application using kubectl and verify that its synced 
directly.

Delete the service file from git repo and notice that its not deleted from destination cluster.

<pre>
controlplane $ kubectl get all -n auto-pruning-demo
NAME                                READY   STATUS    RESTARTS   AGE
pod/guestbook-ui-56c646849b-8ntl8   1/1     Running   0          5m5s

NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/guestbook-ui   ClusterIP   10.100.248.76   <none>        80/TCP    5m5s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/guestbook-ui   1/1     1            1           5m5s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/guestbook-ui-56c646849b   1         1         1       5m5s
controlplane $ 
</pre>

![Web UI](auto purne.PNG)

###   with auto prune enabled
<pre>
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata: 
  name: auto-pruning-demo 
  namespace: argocd
spec: 
  destination: 
    namespace: auto-pruning-demo
    server: "https://kubernetes.default.svc"
  project: default
  source: 
    path: guestbook-with-sub-directories
    repoURL: "https://github.com/rajeswarithota1715/argocd-private-repo-for-demo.git"
    targetRevision: master
    directory:
      recurse: true
  syncPolicy:
    automated:
      prune:  true
    syncOptions:
      - CreateNamespace=true
</pre>

Then, enable the auto pruning in application definition and verify the service is deleted from destination cluster.

![Web UI](autopurne.PNG)

<pre>
   controlplane $ kubectl get all -n auto-pruning-demo
NAME                                READY   STATUS    RESTARTS   AGE
pod/guestbook-ui-56c646849b-8ntl8   1/1     Running   0          24m

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/guestbook-ui   1/1     1            1           24m

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/guestbook-ui-56c646849b   1         1         1       24m
</pre>

