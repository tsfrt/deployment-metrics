## Application Deployment Metrics

A basic observability solution for tracking application deployment frequency that utilizes the Tanzu supported packages for Grafana and Prometheus.  This solution leverages the keptn-lifecycle-toolkit to collect metrics on the frequency, longevity, and reliability of native kubernetes deployments.

### Prerequisites

- Cert Manager installed

Tanzu Observability/Monitoring Installed

[Docs](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/2.2/using-tkg-22/workload-packages-monitoring.html)

Configure monitoiring and metrics packages as documented.  The following examples are minimally configured for ingress (check docs for the latest version and config of packages):

```yaml

ingress:
  enabled: true
  virtual_host_fqdn: "prom.dora.h2o-2-18171.h2o.vmware.com"
  prometheus_prefix: "/"
  alertmanager_prefix: "/alertmanager/"

```

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
#In this example, there is a `metrics`` project created in harbor.  Alter your tag based on your registry solution

imgpkg push -b harbor.build.h2o-2-18171.h2o.vmware.com/metrics/dora-metrics:1.0.3 -f metrics-package/

imgpkg push -b  harbor.build.h2o-2-18171.h2o.vmware.com/metrics/dora-package-repo:1.0.0 -f metrics-package-repo/ 

```

very that your imgpkg bundles are present in your registry