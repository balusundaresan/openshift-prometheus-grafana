```
oc new-project monitoring
oc new-app docker.io/hawkular/hawkular-grafana-datasource
oc expose svc hawkular-grafana-datasource
oadm pod-network make-projects-global monitoring
oadm policy add-cluster-role-to-user cluster-reader system:serviceaccount:monitoring:default


oc create configmap grafana-config --from-file=grafana-config/grafana
oc create configmap grafana-import-dashboards --from-file=grafana-config/grafana-dashboards/

oc volume --add dc/hawkular-grafana-datasource --name config-volume     -t configmap --configmap-name  grafana-config -m /etc/grafana --overwrite
oc volume --add dc/hawkular-grafana-datasource --name dashboards-volume -t configmap --configmap-name  grafana-import-dashboards  -m /var/lib/grafana/dashboards --overwrite


oc create -f https://raw.githubusercontent.com/hawkular/hawkular-openshift-agent/master/deploy/openshift/hawkular-openshift-agent-configmap.yaml -n monitoring
oc process -f https://raw.githubusercontent.com/hawkular/hawkular-openshift-agent/master/deploy/openshift/hawkular-openshift-agent.yaml | oc create -n monitoring -f -
oc adm policy add-cluster-role-to-user hawkular-openshift-agent system:serviceaccount:monitoring:hawkular-openshift-agent



oc new-app prom/prometheus
oc annotate svc prometheus prometheus.io/scrape='true'

oc create -f prometheus-config/prometheus-env.yaml
oc create -f prometheus-config/prometheus-configmap.yaml
oc create configmap prometheus-rules --from-file=prometheus-config/prometheus-rules
oc volume --add dc/prometheus --name config-volume -t configmap --configmap-name  prometheus-configmap -m /etc/prometheus --overwrite
oc volume --add dc/prometheus --name rules-volume  -t configmap --configmap-name  prometheus-rules     -m /etc/prometheus-rules --overwrite
oc env dc prometheus  --from=configmap/prometheus-env

oc create serviceaccount node-exporter
oadm policy add-scc-to-user privileged system:serviceaccount:monitoring:node-exporter


oc new-app prom/alertmanager
oc annotate svc alertmanager prometheus.io/scrape='true'
oc annotate svc alertmanager prometheus.io/path='/alertmanager/metrics'
oc create configmap alertmanager-templates --from-file=alertmanager-config/alertmanager-templates
oc create -f alertmanager-config/alertmanager-configmap.yaml
oc volume --add dc/alertmanager --name config-volume     -t configmap --configmap-name  alertmanager-configmap -m /etc/alertmanager           --overwrite
oc volume --add dc/alertmanager --name templates-volume  -t configmap --configmap-name  alertmanager-templates -m /etc/alertmanager-templates --overwrite






```
