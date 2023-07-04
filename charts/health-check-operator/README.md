# health-check-operator
---
This `helm chart` installs the Native Health Check Operator for Kubernetes made by (Cloudders)[https://cloudders.com] and its CRDs.

_Note: this software is still in beta. Any issues can be reported to the Cloudders team at info[@]cloudders.com_
---

## Getting started

### Prerequisites
- Kubernetes 1.17+
- Helm 3+
---
### Add the Helm repository

In order to add the operator helm repository you need to execute the following commands:

```
kubectl create namespace _your_favourite_namespace_name_
helm repo add cloudders https://helm.cloudders.com/
helm repo update
```
---
### Installation of the chart

Installation can be achieved with the following command:

```
helm install health-check-operator cloudders/health-check-operator
```
---
### Uninstalling the chart
```
helm uninstall health-check-operator
```
---

## Configuration

### HealthCheck CRD

To create new health check you can use the following CRD example and modify it as per your needs:

```
apiVersion: check.cloudders.com/v1alpha1
kind: HealthCheck
metadata:
  name: myFirstHealthCheck
spec:
  ## URL of the destination endpoint to be monitored
  url: "https://myAmazingStore.com"
  ## Test interval for the health check (in seconds)
  ## Beware that lowering this interval under 30 seconds
  ## might cause the check to be rate limited from the destination server
  testInterval: 60
  ## Optional SSL certificate validity check
  ## Alarm will be triggered if the SSL certificate 
  ## is set to expire in less than 10 days
  sslCheck: 
    expiration: 10
  ## Search for Keyword (mandatory)
  ## The searchFor: option accepts string and expression
  ## The expression is regex that can be used to parse part of the reponse body
  searchFor:
    expression: ".*K"
```

### NotificationRule CRD

In order to create `alarm` (NotificationRule) for your checks you need to apply similar CRD to your Kubernetes cluster.

```
apiVersion: check.cloudders.com/v1alpha1
kind: NotificationRule
metadata:
  name: myFirstNotificationRule
spec:
  ## Check CRD support multiple or single checks
  ##
  ## You can use one check like in the example
  ## or multiple like this:
  ##
  ## checks:
  ## - "check1"
  ## - "check2"
  checks:
    - "myFirstHealthCheck"
  slack:
    ## Slack webhook secret to be used
    webhook: "https://..."
    ## Slack username to use
    username: "controller"
    ## Channel where to send the alerts to
    channel: "#general"
    ## Message can support multiple text formats
    ## Normal text without any variables and such with variables, emojies and markdown
    ## Slack blocks are supported as well
    ## Here are some examples of normal text message:
    ##
    ## Alert for *%check%* was triggered at %triggerTime%.
    ## 
    ## and below is example using Slack blocks
    message: |
      {
          "color": "#FF0000",
          "blocks": [
            {
              "type": "section",
              "text": {
                "type": "mrkdwn",
                "text": "Alert for check *%check%* :warning:"
              }
            },
            {
              "type": "section",
              "fields": [
                {
                  "type": "mrkdwn",
                  "text": "*Average response time: *\n%response_time%"
                },
                {
                  "type": "mrkdwn",
                  "text": "*Triggered at: *\n%triggerTime%"
                }
              ]
            },
            {
              "type": "section",
              "fields": [
                {
                  "type": "mrkdwn",
                  "text": "*Current response code is: *\n%response_code%"
                },
                {
                  "type": "mrkdwn",
                  "text": "*Search type used is: *\n%search_type%"
                }
              ]
            }
          ]
      }
  # Time to wait before sending the alert after a check has failed
  # The time is in minutes and its handy parameter in case of short outages or hiccups
  waitBeforeSend: 1
  # Repeat the alert notification every (n) minutes
  repeatAfter: 1
```

---

## Status of HealthChecks

You can get the status of your health checks simply by running:
```
kubectl get healthchecks
```

## Status of NotificationRules

The notification rules also have status like the health checks. You can get the status by executing:
```
kubectl get notificationrules
```