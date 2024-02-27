**ArgoCD Application**

• Application is a Kubernetes resource object 
representing a deployed application instance in an 
environment.

• It is defined by two key pieces of information:

  • Source: reference to the desired state in Git (repository, 
revision, path)

  • Destination: reference to the target cluster and 
namespace.

Applications can be created using below options
1. Declaratively “Yaml”. (Recommended)
2. Web UI
3. CLI

***Declaratively***

<pre>
controlplane $ cat /practice/application.yaml 
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
    repoURL: "https://github.com/mabusaa/argocd-example-apps.git"
    targetRevision: master
  syncPolicy:
    syncOptions:
      - CreateNamespace=true

controlplane $ kubectl apply -f /practice/application.yaml 
application.argoproj.io/guestbook created

controlplane $ kubectl get application -n argocd
NAME        SYNC STATUS   HEALTH STATUS
guestbook   OutOfSync     Missing  
</pre>

***CLI***
<pre>
  argocd app create app-1  --repo  https://github.com/mabusaa/argocd-example-apps.git --path guestbook  --dest-server https://kubernetes.default.svc  --dest-namespace default
  
</pre>

<pre>
  controlplane $ argocd app create app-2 --repo https://github.com/mabusaa/argocd-example-apps.git --path guestbook --revision master --dest-server https://kubernetes.default.svc --dest-namespace app-2 --sync-option CreateNamespace=true
application 'app-2' created

  controlplane $ argocd app list
NAME   CLUSTER                         NAMESPACE  PROJECT  STATUS     HEALTH   SYNCPOLICY  CONDITIONS  REPO                                                PATH       TARGET
app-2  https://kubernetes.default.svc  app-2      default  OutOfSync  Missing  <none>      <none>      https://github.com/mabusaa/argocd-example-apps.git  guestbook  master
controlplane $ argocd app sync app-2
TIMESTAMP                  GROUP        KIND   NAMESPACE                  NAME    STATUS    HEALTH        HOOK  MESSAGE
2024-02-27T03:22:52+00:00            Service       app-2          guestbook-ui  OutOfSync  Missing              
2024-02-27T03:22:52+00:00   apps  Deployment       app-2          guestbook-ui  OutOfSync  Missing              
2024-02-27T03:22:55+00:00          Namespace                             app-2   Running   Synced              namespace/app-2 created
2024-02-27T03:22:56+00:00            Service       app-2          guestbook-ui    Synced  Healthy              
2024-02-27T03:22:56+00:00            Service       app-2          guestbook-ui    Synced   Healthy              service/guestbook-ui created
2024-02-27T03:22:56+00:00   apps  Deployment       app-2          guestbook-ui  OutOfSync  Missing              deployment.apps/guestbook-ui created
2024-02-27T03:22:56+00:00   apps  Deployment       app-2          guestbook-ui    Synced  Progressing              deployment.apps/guestbook-ui created

Name:               app-2
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          app-2
URL:                http://172.30.1.2:32073/applications/app-2
Repo:               https://github.com/mabusaa/argocd-example-apps.git
Target:             master
Path:               guestbook
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        Synced to master (93860ce)
Health Status:      Progressing

Operation:          Sync
Sync Revision:      93860cefec473c343718a38c99a2e099cc40d209
Phase:              Succeeded
Start:              2024-02-27 03:22:52 +0000 UTC
Finished:           2024-02-27 03:22:56 +0000 UTC
Duration:           4s
Message:            successfully synced (all tasks run)

GROUP  KIND        NAMESPACE  NAME          STATUS   HEALTH       HOOK  MESSAGE
       Namespace              app-2         Running  Synced             namespace/app-2 created
       Service     app-2      guestbook-ui  Synced   Healthy            service/guestbook-ui created
apps   Deployment  app-2      guestbook-ui  Synced   Progressing        deployment.apps/guestbook-ui created

controlplane $ kubectl get all -n app-2
NAME                                READY   STATUS    RESTARTS   AGE
pod/guestbook-ui-56c646849b-8dz4p   1/1     Running   0          43s

NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/guestbook-ui   ClusterIP   10.102.114.11   <none>        80/TCP    44s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/guestbook-ui   1/1     1            1           43s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/guestbook-ui-56c646849b   1         1         1       43s
</pre>
