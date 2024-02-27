###  Tools supported

1.  Helm charts
2.   Kustomize application
3.   Directory of Yaml files
4.   Jsonnet


***Directory of Yaml***

• ArgoCD provides the below as options 

• Recursive: include all files in sub-directories. 

• Jsonnet

  • External Vars : list of external variables for Jsonnet.
  
  • Top level Arguments.

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
      jsonnet:
        extVars:
        - name: service
        value: “internal”
</pre>

***Heal Charts***

Helm Applications an be deployed from two sources

• Git Repo.

• Helm Repo.

####  ArgoCD provies the below for options 

• Release name.

• Values files.

• Parameters.

• File parameters.

• Values as block file

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
    valuesFiles: # can set multi values files, (defaults to values.yaml in source repo)
    - values-prod.yaml
    parameters: # override any values in a values.yaml
    - name: “service.type”
      value: “LoadBalancer”
    - name: “image.tag”
      value: “v2”
    fileParameters: # set parameter values from a file
    - name: config
      value: files/config.json
  values: |
    ingress:
      enabled: true
    path: /
    hosts:
      - mydomain.example.com
</pre>

***kustomize***

ArgoCD provides the below for options 

• Name prefix: appended to resources. 

• Name suffix: appended to resources. 

• Images : to override images.

• Common labels: set labels on all resources.

• Common annotations: set annotations on all resources.

• Version: explicitly set kustomize version.

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
      nameSuffix: -staging # adds a suffix to all resources names
      commonLabels:
        app: demo
      images: 
      - gcr.io/heptio-images/ks-guestbook-demo:0.2
      commonAnnotations: 
        app: demo
        appVersion: 1.0
</pre>
