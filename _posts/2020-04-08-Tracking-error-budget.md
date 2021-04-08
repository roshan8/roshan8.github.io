### What's the error budget and why is it important to track it?

<p> Most organizations today track their product SLO’s to avoid being liable for breach of SLAs (Service level agreements). In case any SLO violation occurs, they will be under obligation to pay something in return for breach of contract. Once the SLO for their product has been defined a corresponding error budget will be calculated based on that number. For example, If 99.99% is the SLO, then the error budget will be 52.56 mins in a year. That’s the amount of downtime that the product may have in a year without breaching the SLO. This error budget tracker seeks to provide a simple and effective way to keep track of the error budget burn rate without the hassle of configuring and aggregating multiple data sources.</p>   

<p>Once companies agree on the SLO, they need to pick the most relevant SLI’s(service level indicators). Any violation of these SLI’s will be considered as downtime and the duration of downtime will be deducted from the error budget. For example, a payment gateway product might have the following SLI’s.</p>
- Latency on p95 for requests
- ErrorRates
- Payment failures etc


### Why is it challenging for many companies to track error budgets at the moment?

<p>Usually organizations use a mix of tools to monitor/track these SLI’s (For eg: latency-related SLI’s generally tracked in APM’s such as Newrelic while other SLI’s tracked in monitoring tools such as Prometheus, Datadog). That makes it hard to keep track of the error budget in one centralized location.</p>

<p>Suppose, if we are tracking everything in Prometheus, many companies might have a metrics retention period of less than six months if they chose not to set up Thanos/Cortex, federation rules, or capacity planning for metrics storage.</p>

<p>Next comes the problem of false positives - even if you are tracking something in Prometheus, it’s hard to flag an event as false positive when the incident is not a genuine SLO violation. Tuning proper thresholds for SLI’s takes time and initially we might end up getting a lot of false positives. Or we might end up changing the threshold after some time and you may want to mark some old violations as false positives to get your error budget back.</p>


### What does the SLO tracker do?
SLO-tracker is a simple but effective way to track error budgets. 
- Users first have to set up their target SLO and the error budget will be allocated based on that.
- It currently supports webhook integration with few monitoring tools(Prometheus, Pingdom, and Newrelic) and whenever it receives an incident it will reduce some error budget.
- If a violation is not caught in your monitoring tool or if our tool doesn’t have integration with the monitoring tool then the incident can be reported manually through the user interface.
- Provides some analytics into SLO violation distribution. (Error budget spent on SLI distribution graph) 
- This tool doesn’t require much MySQL storage since this only stores violations but not every metric.

### How to set this up?
- Clone the [repo](https://github.com/roshan8/slo-tracker)
- Repo already comes with a docker-compose so just run `docker-compose up -d`
- Now set some SLI targets in the SLO tracking tool.
- Copy the SLOtracker webhook to your monitoring tool
  - For prometheus: serverip:8080/webhook/prometheus
  - For Newrelic: serverip:8080/webhook/newrelic
  - For Pingdom: serverip:8080/webhook/pingdom
- Now set up rules in your monitoring tool (For eg: Prometheus)
Alert manager rule to monitor an example SLI: Nginx p99 latency is higher than 3 seconds      

```sh
  - alert: NginxLatencyHigh
    expr: histogram_quantile(0.99, sum(rate(nginx_http_request_duration_seconds_bucket[2m])) by (host, node)) > 3
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: Nginx latency high (instance {{ $labels.instance }})
      description: Nginx p99 latency is higher than 3 seconds\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}
```

```sh
Alert routing based on tags set in checks
          global:
            resolve_timeout: 10m
          route:
            routes:
            - receiver: blackhole
            - receiver: slo-tracker
              group_wait: 10s
              match_re:
                severity: critical
              continue: true
          receivers:
          - name: blackhole
          - name: ‘slo-tracker’
            webhook_config: 
              url: 
'http://ENTERIP:8080/webhook/prometheus'
              send_resolved: true
                
```

- Add different tags if you don’t want to route requests based on the severity tags

### What’s next:
- Add a few more monitoring tool integration
- Tracking multiple product SLO’s
- Create more dashboards for analytics
- Better visualization tools to pinpoint problematic services
