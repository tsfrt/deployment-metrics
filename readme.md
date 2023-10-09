## Application Deployment Metrics

A basic observability solution for tracking application deployment frequency that utilizes the Tanzu supported packages for Grafana and Prometheus.  This solution leverages the keptn-lifecycle-toolkit to collect metrics on the frequency, longevity, and reliability of native kubernetes deployments.

### Prerequisites

- Cert Manager installed

- Tanzu Observability/Monitoring Installed (details below)

[Docs](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/2.2/using-tkg-22/workload-packages-monitoring.html)

Configure monitoiring and metrics packages as documented.  The following examples are minimally configured for ingress (check docs for the latest version and config of packages):

prometheus.yaml
```yaml

ingress:
  enabled: true
  virtual_host_fqdn: "prom.dora.h2o-2-18171.h2o.vmware.com"
  prometheus_prefix: "/"
  alertmanager_prefix: "/alertmanager/"

```

grafana.yaml
```yaml

grafana:
  secret:
    admin_user: YWRtaW4=
    admin_password: YWRtaW4=
ingress:
  enabled: true
  virtual_host_fqdn: grafana.dora.h2o-2-18171.h2o.vmware.com

```


```bash

tanzu package install prometheus \
--package prometheus.tanzu.vmware.com \
--version 2.43.0+vmware.2-tkg.1 \
--values-file prometheus.yaml \
--namespace tkg-system

tanzu package install grafana \
--package grafana.tanzu.vmware.com \
--version 9.5.1+vmware.2-tkg.1 \
--values-file grafana.yaml \
--namespace tkg-system

```

### Life Cycle Toolkit install

Preparing your deployment

```bash

#from within root of project
#you may need to auth to your registy, etc before running
#optionally append --registry-ca-cert-path <your registry CA> for self-signed registry certs
#In this example, there is a `metrics` project created in harbor.  Alter your tag based on your registry solution

imgpkg push -b <your registry>/metrics/dora-metrics:1.0.3 -f metrics-package/

```

verify that your imgpkg bundles are present in your registry

Modify the package repo to use the image reference that you defined above:

```yaml

---
apiVersion: data.packaging.carvel.dev/v1alpha1
kind: Package
metadata:
  name: deployment-metrics.tanzu.vmware.1.0.0
spec:
  refName: deployment-metrics.tanzu.vmware
  version: 1.0.0
  releaseNotes: |
        Initial release of the simple app package
  template:
    spec:
      fetch:
      - imgpkgBundle:
          image: harbor.build.h2o-2-18171.h2o.vmware.com/metrics/dora-metrics:1.0.2 #<--- update this
      template:
      - ytt:
          paths:
          - "keptn/"
          - "jaegar/"
      - kbld:
          paths:
          - ".imgpkg/images.yml"
          - "-"
      deploy:
      - kapp: {}

```

Run kbld to update the images.yml for package

```bash

#for self signed registry certs append --registry-ca-cert-path <your ca cert>

kbld -f packages --imgpkg-lock-output metrics-package-repo/.imgpkg/images.yml  


```
Modify the image reference and apply the package repository to your cluster

```yaml

---
apiVersion: packaging.carvel.dev/v1alpha1
kind: PackageRepository
metadata:
  name deployment-metrics-package-repo
spec:
  fetch:
    imgpkgBundle:
      image: harbor.build.h2o-2-18171.h2o.vmware.com/metrics/deployment-metrics-package-repo:1.0.0 #<--- update this

```

Create the life-cycle metrics namespace

```bash

kubectl create ns lifecycle-metrics

```

Apply the package install in your cluster

```yaml

apiVersion: packaging.carvel.dev/v1alpha1
kind: PackageInstall
metadata:
  name: deployment-metrics-install
spec:
  serviceAccountName: default
  packageRef:
    refName: deployment-metrics.tanzu.vmware
    versionSelection:
      constraints: 1.0.0

```