#  Diffing Customization

Argo CD allows you to optionally ignore differences for certain parts of resources, so Argo CD application will be always in sync status even there are differences between desired state and actual state.
* Argo CD allows you to optionally ignore differences of problematic resources/manifests.
* Examples when you might need diffing customization:
  * A controller or mutating webhook is altering the resources after it was submitted to
  * Kubernetes at runtime in a manner which contradicts Git.
  * A Helm chart is using a template function such as randAlphaNum, which generates different data every time helm template is invoked.
  * There is a bug in the manifest, where it contains extra/unknown fields from the actual K8s spec.
* Diffing customization can be configured at application level or at a system level.

**Ignoring differences Options**

*  Argo CD allows ignoring differences using below options:
    * RFC6902 JSON patches at a specific JSON path (json pointers)
    * JQ path expressions.
    * Ignore differences from fields owned by specific managers defined in metadata.managedFields.
 
<pre>
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata: 
  name: diffing-customization-demo
  namespace: argocd
spec: 
  destination:
    namespace: diffing-customization-demo
    server: "https://kubernetes.default.svc"
  project: default
  source: 
    path: guestbook-with-sub-directories
    repoURL: "https://github.com/mabusaa/argocd-example-apps.git"
    targetRevision: master
    directory:
      recurse: true
  syncPolicy:
    automated: {}
    syncOptions:
      - CreateNamespace=true
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
      - /spec/replicas

</pre>

<pre>
  controlplane $ kubectl get all -n diffing-customization-demo
NAME                                READY   STATUS              RESTARTS   AGE
pod/guestbook-ui-56c646849b-8wx8n   0/1     ContainerCreating   0          5s
pod/guestbook-ui-56c646849b-9zlkd   0/1     ContainerCreating   0          5s

NAME                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/guestbook-ui   ClusterIP   10.100.229.3   <none>        80/TCP    5s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/guestbook-ui   0/2     2            0           5s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/guestbook-ui-56c646849b   2         2         0       5s
controlplane $ 
controlplane $ kubectl scale deployment/guestbook-ui -n diffing-customization-demo --replicas=4
deployment.apps/guestbook-ui scaled
controlplane $ kubectl get all -n diffing-customization-demo
NAME                                READY   STATUS    RESTARTS   AGE
pod/guestbook-ui-56c646849b-2t4rp   1/1     Running   0          44s
pod/guestbook-ui-56c646849b-8wx8n   1/1     Running   0          102s
pod/guestbook-ui-56c646849b-9zlkd   1/1     Running   0          102s
pod/guestbook-ui-56c646849b-qk4q6   1/1     Running   0          44s

NAME                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/guestbook-ui   ClusterIP   10.100.229.3   <none>        80/TCP    102s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/guestbook-ui   4/4     4            4           102s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/guestbook-ui-56c646849b   4         4         4       102s
controlplane $ 
</pre>

Even after increasing the replicas in kubernetes cluster the argoCD application still will bein sync

![diff](https://github.com/rajeswarithota1715/ArgoCD/blob/e8a2f54af0d9f751d7eaf4633edd6ed6b55b4366/diff.PNG)
