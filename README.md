# helm-adopt-testing

This is a test of adopting resources for which https://github.com/helm/helm/pull/7649 provides a fix, which is available from 3.2.0

# ahibbert

This chart is called ahibbert (this is a namespace), it has minio as a dependency. This will have a persistent volume and will serve files.

To create this:

* Move to directory
```
cd ahibbert
```
* Update dependencies
```
helm dependency update
```
* Install and send output somewhere
```
helm upgrade ahibbert . --namespace ahibbert --install --debug -f values.yaml > resources-created
```
* Output will show resources created
* Once running port forward to run the UI
```
$ kubectl -n ahibbert port-forward ahibbert-minio-cdbb99c95-nb2t6 9001:9001
```
* Login with the access key and secret access key
* In minio create a bucket
* Upload a file
* Overwrite release-name annotations
```
$ kubectl -n ahibbert annotate --overwrite serviceaccount,secret,configmap,pvc,role,rolebinding,svc,deploy -l chart=minio-8.0.10,release=ahibbert meta.helm.sh/release-name=minio
serviceaccount/ahibbert-minio annotated
serviceaccount/ahibbert-minio-update-prometheus-secret annotated
secret/ahibbert-minio annotated
configmap/ahibbert-minio annotated
persistentvolumeclaim/ahibbert-minio annotated
role.rbac.authorization.k8s.io/ahibbert-minio-update-prometheus-secret annotated
rolebinding.rbac.authorization.k8s.io/ahibbert-minio-update-prometheus-secret annotated
service/ahibbert-minio annotated
deployment.apps/ahibbert-minio annotated
```

# Minio

* Helm uses helpers for naming stuff (e.g. for minio - https://github.com/minio/charts/blob/master/minio/templates/_helpers.tpl). Resources for the `ahibbert` release are named using the jinja2 `{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" -}}` meaning that it is currently `ahibbert-minio`. In the new chart as the release is called `minio` objects would also be called this as of the code:
```
{{- if contains $name .Release.Name -}}
{{- .Release.Name | trunc 63 | trimSuffix "-" -}}
```
This can be overrided by using `fullnameOverride` so we'll need to add this as `ahibbert-minio`
* Run helm upgrade
```
helm upgrade minio . --namespace ahibbert --install --debug -f values.yaml
```
* Given that some fields are immutable this results in the following:
```
$ helm upgrade minio . --namespace ahibbert --install --debug -f values.yaml
history.go:56: [debug] getting history for release minio
upgrade.go:123: [debug] preparing upgrade for minio
upgrade.go:131: [debug] performing update for minio
upgrade.go:303: [debug] creating upgraded release for minio
client.go:209: [debug] checking 9 resources for changes
client.go:492: [debug] Looks like there are no changes for ServiceAccount "ahibbert-minio-update-prometheus-secret"
client.go:492: [debug] Looks like there are no changes for ServiceAccount "ahibbert-minio"
client.go:492: [debug] Looks like there are no changes for Secret "ahibbert-minio"
client.go:492: [debug] Looks like there are no changes for ConfigMap "ahibbert-minio"
client.go:492: [debug] Looks like there are no changes for PersistentVolumeClaim "ahibbert-minio"
client.go:492: [debug] Looks like there are no changes for Role "ahibbert-minio-update-prometheus-secret"
client.go:492: [debug] Looks like there are no changes for RoleBinding "ahibbert-minio-update-prometheus-secret"
client.go:492: [debug] Looks like there are no changes for Service "ahibbert-minio"
client.go:241: [debug] error updating the resource "ahibbert-minio":
	 cannot patch "ahibbert-minio" with kind Deployment: Deployment.apps "ahibbert-minio" is invalid: spec.selector: Invalid value: v1.LabelSelector{MatchLabels:map[string]string{"app":"minio", "release":"minio"}, MatchExpressions:[]v1.LabelSelectorRequirement(nil)}: field is immutable
upgrade.go:369: [debug] warning: Upgrade "minio" failed: cannot patch "ahibbert-minio" with kind Deployment: Deployment.apps "ahibbert-minio" is invalid: spec.selector: Invalid value: v1.LabelSelector{MatchLabels:map[string]string{"app":"minio", "release":"minio"}, MatchExpressions:[]v1.LabelSelectorRequirement(nil)}: field is immutable
Error: UPGRADE FAILED: cannot patch "ahibbert-minio" with kind Deployment: Deployment.apps "ahibbert-minio" is invalid: spec.selector: Invalid value: v1.LabelSelector{MatchLabels:map[string]string{"app":"minio", "release":"minio"}, MatchExpressions:[]v1.LabelSelectorRequirement(nil)}: field is immutable
helm.go:88: [debug] cannot patch "ahibbert-minio" with kind Deployment: Deployment.apps "ahibbert-minio" is invalid: spec.selector: Invalid value: v1.LabelSelector{MatchLabels:map[string]string{"app":"minio", "release":"minio"}, MatchExpressions:[]v1.LabelSelectorRequirement(nil)}: field is immutable
helm.sh/helm/v3/pkg/kube.(*Client).Update
	helm.sh/helm/v3/pkg/kube/client.go:254
helm.sh/helm/v3/pkg/action.(*Upgrade).performUpgrade
	helm.sh/helm/v3/pkg/action/upgrade.go:317
helm.sh/helm/v3/pkg/action.(*Upgrade).Run
	helm.sh/helm/v3/pkg/action/upgrade.go:132
main.newUpgradeCmd.func2
	helm.sh/helm/v3/cmd/helm/upgrade.go:155
github.com/spf13/cobra.(*Command).execute
	github.com/spf13/cobra@v1.1.3/command.go:852
github.com/spf13/cobra.(*Command).ExecuteC
	github.com/spf13/cobra@v1.1.3/command.go:960
github.com/spf13/cobra.(*Command).Execute
	github.com/spf13/cobra@v1.1.3/command.go:897
main.main
	helm.sh/helm/v3/cmd/helm/helm.go:87
runtime.main
	runtime/proc.go:225
runtime.goexit
	runtime/asm_amd64.s:1371
UPGRADE FAILED
main.newUpgradeCmd.func2
	helm.sh/helm/v3/cmd/helm/upgrade.go:157
github.com/spf13/cobra.(*Command).execute
	github.com/spf13/cobra@v1.1.3/command.go:852
github.com/spf13/cobra.(*Command).ExecuteC
	github.com/spf13/cobra@v1.1.3/command.go:960
github.com/spf13/cobra.(*Command).Execute
	github.com/spf13/cobra@v1.1.3/command.go:897
main.main
	helm.sh/helm/v3/cmd/helm/helm.go:87
runtime.main
	runtime/proc.go:225
runtime.goexit
	runtime/asm_amd64.s:1371
```
* Fields in some resources will be immutable
* Edit the deployment manually and change the labels `release: ahibbert` to `release: minio`, this won't be possible but your yaml will be saved. Delete the deployment and then reapply the yaml to create the deployment
```
$ kubectl -n ahibbert edit deploy ahibbert-minio
error: deployments.apps "ahibbert-minio" is invalid
A copy of your changes has been stored to "/var/folders/d4/07vjpdq934d_9g037zy73md9mh0zf6/T/kubectl-edit-1129265674.yaml"
error: Edit cancelled, no valid changes were saved.
ELSLAPM-404211:minio hibberta$ kubectl -n ahibbert delete deploy ahibbert-minio
deployment.apps "ahibbert-minio" deleted
ELSLAPM-404211:minio hibberta$ kubectl apply -f /var/folders/d4/07vjpdq934d_9g037zy73md9mh0zf6/T/kubectl-edit-1129265674.yaml
deployment.apps/ahibbert-minio created
```
* Port forward to the new pod
```
$ kubectl -n ahibbert port-forward ahibbert-minio-875db87bb-t4tdj 9001:9001
Forwarding from 127.0.0.1:9001 -> 9001
Forwarding from [::1]:9001 -> 9001
```
* Browse the UI and you should see the same data there 

# ahibbert

Annotating with this may be necessary before:

```
https://helm.sh/docs/howto/charts_tips_and_tricks/#tell-helm-not-to-uninstall-a-resource
```
However not sure if that will be rolled back when helm is run

THIS BIT SHOULD NOT BE DONE.

* It is now necessary to remove this from the `ahibbert` release
* Add a `requirements.yaml` file in the `ahibbert` folder with the text:
```
dependencies:
  - name: minio
    version: 8.0.10
    condition: minio.enabled
```
* Add to the `values.yaml` the `enabled: false` under `minio:`
* When you run helm now nothing will change:
```
$ helm upgrade ahibbert . --namespace ahibbert --install --debug -f values.yaml
history.go:56: [debug] getting history for release ahibbert
load.go:120: Warning: Dependencies are handled in Chart.yaml since apiVersion "v2". We recommend migrating dependencies to Chart.yaml.
upgrade.go:123: [debug] preparing upgrade for ahibbert
upgrade.go:131: [debug] performing update for ahibbert
upgrade.go:303: [debug] creating upgraded release for ahibbert
client.go:209: [debug] checking 0 resources for changes
client.go:258: [debug] Deleting "ahibbert-minio-update-prometheus-secret" in ahibbert...
client.go:258: [debug] Deleting "ahibbert-minio" in ahibbert...
client.go:258: [debug] Deleting "ahibbert-minio" in ahibbert...
client.go:258: [debug] Deleting "ahibbert-minio" in ahibbert...
client.go:258: [debug] Deleting "ahibbert-minio" in ahibbert...
client.go:258: [debug] Deleting "ahibbert-minio-update-prometheus-secret" in ahibbert...
client.go:258: [debug] Deleting "ahibbert-minio-update-prometheus-secret" in ahibbert...
client.go:258: [debug] Deleting "ahibbert-minio" in ahibbert...
client.go:258: [debug] Deleting "ahibbert-minio" in ahibbert...
upgrade.go:138: [debug] updating status for upgraded release for ahibbert
Release "ahibbert" has been upgraded. Happy Helming!
NAME: ahibbert
LAST DEPLOYED: Thu Nov 11 20:11:25 2021
NAMESPACE: ahibbert
STATUS: deployed
REVISION: 8
TEST SUITE: None
USER-SUPPLIED VALUES:
minio:
  DeploymentUpdate:
    maxSurge: 100%
    maxUnavailable: 0
    type: RollingUpdate
  StatefulSetUpdate:
    updateStrategy: RollingUpdate
  accessKey: admin123A
  clusterDomain: cluster.local
  configPath: /root/.minio/
  configPathmc: /root/.mc/
  enabled: false
  existingSecret: ""
  extraArgs:
  - --console-address ':9001'
  image:
    pullPolicy: IfNotPresent
    repository: minio/minio
    tag: latest
  mcImage:
    pullPolicy: IfNotPresent
    repository: minio/mc
    tag: latest
  mode: standalone
  mountPath: /export
  persistence:
    accessMode: ReadWriteOnce
    enabled: true
    size: 10Gi
    storageClass: gp2
    subPath: ""
  priorityClassName: ""
  replicas: 1
  secretKey: admin123A
  service:
    annotations: {}
    nodePort: 31311
    port: 9000
    type: NodePort
  tls:
    certSecret: ""
    enabled: false
    privateKey: private.key
    publicCrt: public.crt

COMPUTED VALUES:
minio:
  DeploymentUpdate:
    maxSurge: 100%
    maxUnavailable: 0
    type: RollingUpdate
  StatefulSetUpdate:
    updateStrategy: RollingUpdate
  accessKey: admin123A
  clusterDomain: cluster.local
  configPath: /root/.minio/
  configPathmc: /root/.mc/
  enabled: false
  existingSecret: ""
  extraArgs:
  - --console-address ':9001'
  image:
    pullPolicy: IfNotPresent
    repository: minio/minio
    tag: latest
  mcImage:
    pullPolicy: IfNotPresent
    repository: minio/mc
    tag: latest
  mode: standalone
  mountPath: /export
  persistence:
    accessMode: ReadWriteOnce
    enabled: true
    size: 10Gi
    storageClass: gp2
    subPath: ""
  priorityClassName: ""
  replicas: 1
  secretKey: admin123A
  service:
    annotations: {}
    nodePort: 31311
    port: 9000
    type: NodePort
  tls:
    certSecret: ""
    enabled: false
    privateKey: private.key
    publicCrt: public.crt

HOOKS:
MANIFEST:
```
* The `requirements.yaml` file will need to stay there as long as the release exists
