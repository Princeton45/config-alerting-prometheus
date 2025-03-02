# Alerting Configuration for Kubernetes Microservices Application

## Project Overview
In this project, I configured a comprehensive alerting system for a Kubernetes Microservices application using Prometheus and AlertManager. The system now monitors the application and sends email notifications when CPU usage on a pod exceeds 50% or when a pod can't start.

## Technologies Used
- Prometheus
- Kubernetes
- Linux

## Implementation

### Prometheus Alert Rules Configuration
I configured Prometheus to monitor our application's resources and trigger alerts based on specific conditions:

1. Created an alert rule for CPU usage exceeding 50%
2. Created an alert rule to detect when pods fail to start. If a pod keeps restarting over 5 times, then this will trigger the alert.

Then to actually implement the alert rules in Prometheus, I utilized the Prometheus Operator to create a Custom Resource out of the PrometheusRule CRD (Custom Resource Definition ) for the new alert rules which will tell Prometheus to reload the alert rules.

`alert-rules.yaml`
```yaml
#Custom CRD
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: main-rules
  namespace: monitoring
  labels:
    app: kube-prometheus-stack
    release: monitoring
spec:
  groups:
  - name: main.rules
    rules:
    #First alert rule
    - alert: HostHighCpuLoad
      expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[2m])) * 100) > 50
      for: 2m
      labels:
        severity: warning
        namespace: monitoring
      annotations:
        description: "CPU load on host is over 50%\ Value = {{ $value }}\n Instance = {{ $labels.instance }}\n"
        summary: "Host CPU load high"
    #Second alert rule
    - alert: KubernetesPodCrashLooping
      expr: kube_pod_container_status_restarts_total > 5
      for: 0m
      labels:
        severity: critical
        namespace: monitoring
      annotations:
        description: "Pod {{ $labels.pod }} is crash looping\n Value = {{ $value }}"
        summary: "Kubernetes pod crash looping"
```

![get-alert](https://github.com/Princeton45/config-alerting-prometheus/blob/main/images/get-alert.png)

![alert1](https://github.com/Princeton45/config-alerting-prometheus/blob/main/images/alert1.png)

Next I simulated a CPU load in the EKS cluster to trigger the `HostHighCpuLoad` alert

I used the `containerstack/cpustress` Docker image and created a pod out of it to simulate the CPU load.

`kubectl run cpu-test --image=containerstack/cpustress -- --cpu 4 --timeout 30s --metrics-brief`

Now the `HostHighCpuLoad` alert is in Pending state

![pending](https://github.com/Princeton45/config-alerting-prometheus/blob/main/images/pending.png)

Since the load has been above 50 percent for over 2 minutes (according to the alert rule), it will then go into firing state which will trigger the alert and notify the Operations team (once AlertManager is configured).

![firing](https://github.com/Princeton45/config-alerting-prometheus/blob/main/images/firing.png)


### AlertManager Setup
I installed and configured AlertManager to process the alerts from Prometheus:

1. Deployed AlertManager on our Kubernetes cluster
2. Created configuration files to define alert routing
3. Set up an email receiver to notify the team

The Prometheus Operator handles configurations for Alert Manager, so I created a custom resource from the `AlertManagerConfig` CRD

`alert-manager-config.yaml`
```yaml
#Custom CRD
apiVersion: monitoring.coreos.com/v1alpha1
kind: AlertmanagerConfig
metadata:
  name: main-rules-alert-config
  namespace: monitoring
spec:
  #Configuring the route
  route:
    receiver: 'email'
    repeatInterval: 30m
    routes:
    - matchers:
      - name: alertname
        value: HostHighCpuLoad
    - matchers:
      - name: alertname
        value: KubernetesPodCrashLooping
      repeatInterval: 10m
  #Configuring email receiver
  receivers:
  - name: 'email'
    emailConfigs:
    - to: 'princetonabdulsalam25@gmail.com'
      from: 'princetonabdulsalam25@gmail.com'
      smarthost: 'smtp.gmail.com:587'
      authUsername: 'princetonabdulsalam25@gmail.com'
      authIdentity: 'princetonabdulsalam25@gmail.com'
      authPassword: 
        name: gmail-auth
        key: password
```

I also created `email-secret.yaml` which contains a base64 encoded secret of the authPassword which is not checked into this git repository

Now in the Alertmanager status at `http://127.0.0.1:9093/#/status`, we can see the configuration added:

![alert-man](https://github.com/Princeton45/config-alerting-prometheus/blob/main/images/alert-man.png)

![routes](https://github.com/Princeton45/config-alerting-prometheus/blob/main/images/routes.png)


### Email Notification Configuration
The email notification system now:
- Sends detailed alerts when thresholds are breached
- Includes relevant metrics in the notification
- Delivers messages to our team's email address

![Sample Alert Email Screenshot](suggested-image-alert-email.png)

## Results
Our monitoring infrastructure now automatically alerts the team whenever:
- CPU utilization exceeds 50% on any node
- Pods fail to start or enter a crash loop
- Other critical conditions occur that might impact application performance

This proactive monitoring approach allows us to address issues before they affect users and maintain high service availability.

## Future Improvements
- Add Slack notification channel
- Implement PagerDuty integration for urgent alerts
- Create more granular alerting rules for memory and network metrics

![Monitoring Dashboard Screenshot](suggested-image-dashboard.png)