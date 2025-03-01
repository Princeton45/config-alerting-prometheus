# Alerting Configuration for Kubernetes Microservices Application

## Project Overview
In this project, I configured a comprehensive alerting system for a Kubernetes Microservices application using Prometheus and AlertManager. The system now monitors the  application and sends email notifications when CPU usage on a pod exceed 50% or when a pod can't start.

## Technologies Used
- Prometheus
- Kubernetes
- Linux

## Implementation

### Prometheus Alert Rules Configuration
I configured Prometheus to monitor our application's resources and trigger alerts based on specific conditions:

1. Created alert rules for CPU usage exceeding 50%
2. Set up rules to detect when pods fail to start

![Prometheus Alert Rules Configuration Screenshot](suggested-image-prometheus-rules.png)

### AlertManager Setup
I installed and configured AlertManager to process the alerts from Prometheus:

1. Deployed AlertManager on our Kubernetes cluster
2. Created configuration files to define alert routing
3. Set up an email receiver to notify the team

![AlertManager Configuration Screenshot](suggested-image-alertmanager-config.png)

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