## Application Deployment Metrics

A basic observability solution for tracking application deployments

###Prerequisites

- Cert Manager installed

Tanzu Observability/Monitoring Installed
(Docs)[https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/2.2/using-tkg-22/workload-packages-monitoring.html]

Configure monitoiring and metrics packages as documented.  The following examples are minimally configured for ingress:

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