
# Instrumenting a python application
The goal of this poc is to show how easy it can be to start instrumenting application that we develop and for which we own the code.

For this POC I decided to monitor the AYS api server.
The metrics we are going to monitor are:
- requests rate for each endpoint
- duration of requests for each endpoint in second
- number of jobs, succeeded and failed
- duration of jobs

The first 2 metrics are going to be really easy to expose since sanic has a small plugins that does everything for us: https://github.com/dkruchinin/sanic-prometheus
The way it works is it add a middleware that count the number of requests and the time each requests takes. It then expose these metrics as `/metrics` endpoint on the http server.

Here the diff of the change I made for this step:
```diff
diff --git a/main.py b/main.py
index c92006c..c36556a 100755
--- a/main.py
+++ b/main.py
@@ -10,6 +10,7 @@ import click
 import logging
 from js9 import j
 from JumpScale9AYS.ays.server.app import app as sanic_app
+from sanic_prometheus import monitor

 sanic_app.config['REQUEST_TIMEOUT'] = 3600

@@ -68,6 +69,8 @@ def main(host, port, log, dev):
     async def stop_ays(sanic, loop):
         await j.atyourservice.server._stop()

+    # expose prometheus metrics
+    monitor(sanic_app).expose_endpoint()
     # start server
     sanic_app.run(debug=debug, host=host, port=port, workers=1)
```

For the last 2 metrics, number of jobs and duration of the jobs, will add some monitornig in the Job class, like this:

```diff
diff --git a/JumpScale9AYS/jobcontroller/Job.py b/JumpScale9AYS/jobcontroller/Job.py
index 34ed98d..9191279 100755
--- a/JumpScale9AYS/jobcontroller/Job.py
+++ b/JumpScale9AYS/jobcontroller/Job.py
@@ -11,7 +11,12 @@ import logging
 import traceback
 from collections import MutableMapping
 import time
+from prometheus_client import Counter, Histogram, Gauge

+METRICS = {}
+METRICS['JOB_COUNT'] = Counter("ays_job_count", "AYS job count", ['status'])
+METRICS['JOB_LATENCY'] = Histogram("ays_job_latency_sec", "AYS job latency histogram", ['status'])
+METRICS['JOB_RUNNING'] = Gauge('ays_job_running', "AYS, number of job currently running")

 colored_traceback.add_hook(always=True)

@@ -22,6 +27,8 @@ def _execute_cb(job, future):
     job: is the job object
     future: future that hold the result of the job execution
     """
+    METRICS['JOB_RUNNING'].dec()
+
     if job._cancelled is True:
         return

@@ -44,8 +51,15 @@ def _execute_cb(job, future):
         # catch CancelledError since it's not an anormal to have some job cancelled
         exception = err
         job.logger.info("{} has been cancelled".format(job))
+        # increase job counter for timeout job
+        METRICS['JOB_COUNT'].labels(status='timeout').inc()
+        METRICS['JOB_LATENCY'].labels(status='timeout').observe(elapsed)

     if exception is not None:
+        # increase job counter for error job
+        METRICS['JOB_COUNT'].labels(status='error').inc()
+        METRICS['JOB_LATENCY'].labels(status='error').observe(elapsed)
+
         # state state of job and run to error
         # this state will be check by RunStep and Run and it will raise an exception
         job.state = 'error'
@@ -70,6 +84,10 @@ def _execute_cb(job, future):
             job.logger.error("{} failed:\n{}".format(job, '\n'.join(tb_lines)))

     else:
+        # increase job counter for succefull job
+        METRICS['JOB_COUNT'].labels(status='ok').inc()
+        METRICS['JOB_LATENCY'].labels(status='ok').observe(elapsed)
+
         # job executed succefully
         job.state = 'ok'
         job.model.dbobj.state = 'ok'
@@ -342,6 +360,8 @@ class Job:

         ex: result, stdout, stderr = await job.execute()
         """
+        METRICS['JOB_RUNNING'].inc()
+
         self._started = time.time()

         # for now use default ThreadPoolExecutor
```

So we declare the counter, histogram and gauge that we are going to use, then based on the status of the job we register the metrics.

Now all the metrics we want to gather are configured we can setup prometheus to start collecting data. The configuration is extremely simple:

```yaml
# my global config
global:
  scrape_interval:     5s # Set the scrape interval to every 5 seconds.
  evaluation_interval: 15s # Evaluate rules every 15 seconds.

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
      monitor: 'codelab-monitor'

scrape_configs:
   # here we configure prometheus to monitor itself
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  # here we configure prometheus to monitor the ays process
  - job_name: 'ays'
    static_configs:
      - targets: ['localhost:6600']

```

Once the configuration is written simply start prometheus with
`/prometheus --config.file=prometheus.yml`


Now all the metrics we want to gather are configured we can start querying prometheus. Here is some example query:

Average duration of a job over the last minute

```PromQL
rate(ays_job_latency_sec_sum [1m]) / rate(ays_job_latency_sec_count [1m])
````

Average http request latency over last 30 minutes:

```PromQL
rate(sanic_request_latency_sec_sum [10m]) / rate(sanic_request_latency_sec_count [10m])
```

Number of job currently running
```PromQL
sum(ays_job_running)
```

#### Final though on this poc
I really like how easy it is to make an application prometheus enabled.

The client are mature and easy to use for Python and Go.

Direct integration between prometheus and grafana remove the need to have another time serie database to maintain. The overall management is way less complex with prometheus then with our current statistic infrastructure. We pass from 3 different processes (influxdb, redis, influxdumper) to manage to 1 (prometheus) and we win alerting support for free.