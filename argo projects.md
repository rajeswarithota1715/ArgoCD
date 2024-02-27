### Why Projects ?

1.  Projects provide a logical grouping of applications.
2.  Access restrictions
      * Useful when ArgoCD is used by multiple teams.
      * Allow only specific sources “trusted git repos”.
      * Allow apps to be deployed into specific clusters and namespaces.
      * Allow specific resources to be deployed “Deployments, Statefulsets .. etc”.
  
3.  Enables you to create a role with set of policies “permissions” to grant access to a project's applications.
     * You can use it to grant CI system a specific access to project applications.(It must be associated with JWT.)
     * You can use it to grant oidc groups a
     * pecific access to project applications
  
4.  ArgoCD creates a default project once you install it.

we can create projects using 3 options

1.  Declarative
2.  CLI
3.  Web UI

***Declarative***

"*" means everything
below project will only allow resources to deploy in ns-1 namespace
<pre>
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: dev-project 
  namespace: argocd
spec:
  description: Dev project
  sourceRepos:
  - '*'

  destinations:
  - namespace: ns-1 
    server: https://kubernetes.default.svc

  clusterResourceWhitelist:
  - group: '*'
    kind: '*'

  namespaceResourceWhitelist:
  - group: '*'
    kind: '*'
</pre>
<pre>
controlplane $ kubectl apply -f /practice/project.yaml
appproject.argoproj.io/dev-project created

controlplane $ kubectl get appproject -n argocd
NAME          AGE
default       4m13s
dev-project   2s

</pre>

try to deploy the following application in dev-project test-1 name space

<pre>
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata: 
  name: guestbook-dev-project 
  namespace: argocd
spec: 
  destination: 
    namespace: test-1
    server: "https://kubernetes.default.svc"
  project: dev-project
  source: 
    path: guestbook
    repoURL: "https://github.com/mabusaa/argocd-example-apps.git"
    targetRevision: master 
  syncPolicy:
    syncOptions:
      - CreateNamespace=true
</pre>
<pre>
controlplane $ kubectl apply -f /practice/application.yaml
application.argoproj.io/guestbook-dev-project created

controlplane $ kubectl get application -n argocd
NAME                    SYNC STATUS   HEALTH STATUS
guestbook-dev-project   Unknown       Unknown
</pre>

the status is showing as unknown , let's describe the application to check the reason
<pre>
controlplane $ kubectl describe application -n argocd
Name:         guestbook-dev-project
Namespace:    argocd
Labels:       <none>
Annotations:  <none>
API Version:  argoproj.io/v1alpha1
Kind:         Application
Metadata:
  Creation Timestamp:  2024-02-27T06:04:39Z
  Generation:          2
  Resource Version:    3096
  UID:                 6768afc2-0bbb-4e15-b7b8-05a3d4d80ebe
Spec:
  Destination:
    Namespace:  test-1
    Server:     https://kubernetes.default.svc
  Project:      dev-project
  Source:
    Path:             guestbook
    Repo URL:         https://github.com/mabusaa/argocd-example-apps.git
    Target Revision:  master
  Sync Policy:
    Sync Options:
      CreateNamespace=true
Status:
  Conditions:
    Last Transition Time:  2024-02-27T06:04:39Z
    Message:               **application destination {https://kubernetes.default.svc test-1} is not permitted in project 'dev-project'**
    Type:                  InvalidSpecError
  Health:
    Status:  Unknown
  Sync:
    Status:  Unknown
Events:
  Type    Reason           Age    From                           Message
  ----    ------           ----   ----                           -------
  Normal  ResourceUpdated  2m19s  argocd-application-controller  Updated sync status:  -> Unknown
  Normal  ResourceUpdated  2m19s  argocd-application-controller  Updated health status:  -> Unknown
controlplane $ 
</pre>

it's clearly informing destination is not permitted in project dev-project

in order to run the application you need to change the namespace to ns-1 and apply the changes 

###  Projecr Role

project role features enables you to create a role with set of policies “permissions” to grant access to a project's applications

<pre>
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: project-with-role 
  namespace: argocd
spec:
  description: project with ci-role
  sourceRepos:
  - '*'

  destinations:
  - namespace: "*"
    server: "*"

  clusterResourceWhitelist:
  - group: '*'
    kind: '*'

  namespaceResourceWhitelist:
  - group: '*'
    kind: '*'

  roles:
  - name: ci-role
    description: Sync privileges for project-with-role
    policies:
    - p, proj:project-with-role:ci-role, applications, sync, project-with-role/*, allow
</pre>
<pre>
controlplane $ kubectl apply -f /practice/project.yaml
appproject.argoproj.io/project-with-role created
controlplane $ 
controlplane $ kubectl get appproject -n argocd
NAME                AGE
default             6m49s
project-with-role   2s
</pre>
**Creating a token**

• Project roles is not useful without generating a JWT.

• Generated tokens are not stored in ArgoCD.

• To create a token using CLI

argocd proj role create-token PROJECT ROLE-NAME

<pre>
controlplane $ argocd proj role create-token project-with-role ci-role
Create token succeeded for proj:project-with-role:ci-role.
  ID: 9dc2ceb6-d579-4d6a-b386-4d01a6d9580b
  Issued At: 2024-02-27T06:32:45Z
  Expires At: Never
  Token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJhcmdvY2QiLCJzdWIiOiJwcm9qOnByb2plY3Qtd2l0aC1yb2xlOmNpLXJvbGUiLCJuYmYiOjE3MDkwMTU1NjUsImlhdCI6MTcwOTAxNTU2NSwianRpIjoiOWRjMmNlYjYtZDU3OS00ZDZhLWIzODYtNGQwMWE2ZDk1ODBiIn0.ytrS2T08cRNKd9m4_qryLWBQf2kf1n34LOcTEKVDP48
controlplane $ 
</pre>

**Using the token in CLI**

• A user can leverage tokens in the cli by either passing them in using the --auth-token flag or setting the ARGOCD_AUTH_TOKEN environment variable.

Ex: argocd cluster list --auth-token token-value
<pre>
  controlplane $ argocd app delete demo --grpc-web --auth-token eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJhcmdvY2QiLCJzdWIiOiJwcm9qOnByb2plY3Qtd2l0aC1yb2xlOmNpLXJvbGUiLCJuYmYiOjE3MDkwMTU1NjUsImlhdCI6MTcwOTAxNTU2NSwianRpIjoiOWRjMmNlYjYtZDU3OS00ZDZhLWIzODYtNGQwMWE2ZDk1ODBiIn0.ytrS2T08cRNKd9m4_qryLWBQf2kf1n34LOcTEKVDP48
Are you sure you want to delete 'demo' and all its resources? [y/n]
yes
FATA[0002] rpc error: code = PermissionDenied desc = permission denied: applications, delete, project-with-role/demo, sub: proj:project-with-role:ci-role, iat: 2024-02-27T06:32:45Z 
</pre>

as you can see above we got permission denied error while tryig to delete the project using auth code , because we don't have permissions to delete the project ,we only have acces to sync the project




  
