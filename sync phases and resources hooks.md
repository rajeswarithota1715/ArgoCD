
![sync phsases](https://github.com/rajeswarithota1715/ArgoCD/blob/6a63921a555a9816edfb47e1e15ab4d0571925ad/sync-phases.PNG)

* You can use phases using resources hooks annotation argocd.argoproj.io/hook on app manifests.
* You can add hook annotation to any manifests in your git repo
* Hooks does not run during selective sync.

**Resources Hook types**
| Hook | Description |
|:------:|:-----------:|
|PreSync |Executes prior to the app manifests
|Sync| Executes after all PreSync hooks successfully completed and healthy.
|PostSync| Executes after all Sync hooks successfully completed and healthy.
|Skip| Argo CD will skip syncing the manifest.
|SyncFail |Executes when the sync operation fails.

**Hook Deletion Policies**

Hooks recourses can be automatically deleted by several options using this **annotation argocd.argoproj.io/hook-delete-policy**

|Deletion policy | Description
|----------------|-------------|
|HookSucceeded| The hook resource is deleted after the hook succeeded.|
|HookFailed| The hook resource is deleted after the hook failed.|
|BeforeHookCreation| Any existing hook resource is deleted before the new one is created (default policy)|



