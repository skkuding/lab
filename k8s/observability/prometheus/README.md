# What is Prometheus?

Prometheus is an open-source monitoring and alerting toolkit to collect and store **metrics**.

## What is kube-prometheus-stack?

kube-prometheus-stack is a collection of Kubernetes manifests, Grafana dashboards, and Prometheus rules combined into a single package to provide a complete monitoring solution for Kubernetes clusters.

It contains the following components:

- Prometheus: for collecting and storing metrics
- Grafana: for visualizing metrics
- Node Exporter: for collecting hardware and OS metrics
- Kube-state-metrics: for collecting Kubernetes object metrics

## Why use kube-prometheus-stack?

kube-prometheus-stack automatically deploys and configures all the components needed for a complete monitoring solution for Kubernetes clusters.
With DaemonSets, it ensures that the Node Exporter is running on every node in the cluster, providing comprehensive hardware and OS metrics.

## Why is Grafana disabled?

We use kube-prometheus-stack because it provides a complete monitoring solution for Kubernetes clusters.
However, in order to separate concerns and manage resources more effectively, we can deploy Grafana as a standalone application.
Grafana is not only for prometheus but also for other data sources, including Loki for logs and Tempo for traces.

## How kube-prometheus-stack and OTel Collector are connected?: ServiceMonitor

kube-prometheus-stack discovers targets by looking for a CRD(Custom Resource Definition), ServiceMonitors in the cluster.
ServiceMonitors inside the kubernetes cluster are deployed by helm chart of kube-prometheus-stack.
However in case of OTel Collector, which we defined through the OpenTelemetryCollector CRD, we need to create a ServiceMonitor manually to enable scraping because it is not automatically discovered by kube-prometheus-stack.

## ProSeed PostgreSQL backup failure alert

`ProseedPostgresBackupFailed` detects a failed `postgres-backup-*` Job in the
`proseed-postgres` namespace and sends the alert to Discord through
Alertmanager. The rule only considers failures completed during the last 24
hours, so retained historical Job objects do not keep sending stale alerts.

### Create the Discord webhook Secret

The webhook URL must never be committed to Git. Create the Secret before
merging or syncing the Prometheus change because Alertmanager mounts it at
startup.

```bash
read -rs "DISCORD_WEBHOOK_URL?Discord webhook URL: "
printf %s "${DISCORD_WEBHOOK_URL}" | \
  kubectl --context lab -n observability create secret generic proseed-discord-webhook \
    --from-file=webhook-url=/dev/stdin \
    --dry-run=client -o yaml | \
  kubectl --context lab apply -f -
unset DISCORD_WEBHOOK_URL
```

### Test the notification

Create a disposable failed Job after Prometheus and Alertmanager are healthy.
It should produce one Discord notification after about 1-2 minutes.

```bash
kubectl --context lab -n proseed-postgres create job postgres-backup-alert-test \
  --image=busybox:1.36 -- /bin/sh -c 'exit 1'

kubectl --context lab -n proseed-postgres get job postgres-backup-alert-test
kubectl --context lab -n proseed-postgres delete job postgres-backup-alert-test
```
