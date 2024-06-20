Jim Johnson, ISU

Were using Solarwinds monitoring, which wasn't great at time series. 
Changed outputs from telegraf to prometheus.

Ansible playbooks are available on GitHub (ISU security office already reviewed): [https://github.com/IllinoisStateUniversity](https://github.com/IllinoisStateUniversity).

Keeper for secrets manager
Mimir for time series database

Deployed primarily with redhat.openshift.k8s module and jinja templates.
Pull from docker hub and apply a config.yaml file.

Had multiple monitoring environments running that were replaced successfully.

## OpenShift
Need a namespace.
Need a service account on the namespace.
For grafana, you need a second credential for cluster admin to deploy the operator into the namespace (unless it is pre-deployed into your namespace). Can use Kubernetes Bearer tokens injected via AAP rather than sending a credential in the playbooks.

## Mimir
Deploy Mimir instance and expose a service/port.
Stores to ceph/s3-style bucket.
ISU doesn't need large/HA capacity yet.
No PVC needed.
Mimir supports HA out of the box more easily than Prometheus.

## Prometheus
The remote write config requires a special header `X-Scope-OrgID: <id>`
Using Prometheus in agent mode to write directly to Mimir, not storing any data in prometheus.
Don't need a PVC when using Prometheus as agent.
Make a config map for Prometheus configuration.
Deploy the deployment object to create pod(s).
Expose the prometheus port/service (for troubleshooting).

## Grafana
Need cluster admin to install operator.
Created an Ansible role to set up ACME certs.
Uses a service account to login to OpenShift.
Grafana does not need a PVC, but customized dashboards cannot be persisted without persistent storage, so a small PVC is helpful.
Grafana config sets a username and password.
Dashboards are part of the operator functionality. Dashboards are configured in a folder, and automatically looped in during creation of container.
Secrets are pulled directly from Keeper, so if Keeper is down, the configs will fail. 
Caching secrets in OpenShift would allow OpenShift to cache them and the playbooks would succeed even if Keeper is unavailable.

Use the same naming in test and prod.
Then when moving workflow to production, everything works.

## Future state
Event Driven Ansible can insert metadata on the dashboards for alerts triggered and playbooks run.
Could fairly easily expand to high availability model with this stack.

