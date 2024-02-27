### Private Git Repos

• Public repos can be used directly in application.

• Private repos needs to be registered in ArgoCD with proper authentication before using it in applications.

• ArgoCD support connecting to private repos using below ways:

  *  HTTPs: using username and password or access token.
  *  SSH: using ssh private key.
  *  GitHub / GitHub Enterprise : GitHub App credentials.
• Private repos credentials are stored in normal k8s secrets.

• You can register repos using declarative approach, cli and web UI.
<pre>
</pre>
![private got repo](https://github.com/rajeswarithota1715/ArgoCD/blob/main/git-private-repo.PNG)  
