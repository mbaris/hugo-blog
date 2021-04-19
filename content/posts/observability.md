---
title: "Observability 101"
date: 2021-04-19T23:50:25+02:00
draft: false
summary: "Observability is a measure of how well internal states of a system can be inferred from knowledge of its external outputs."
---

Systems we build and maintain are becoming more complex and abstract with the increase in popularity of distributed systems, microservices, and cloud providers. With these new concepts, our view of failures has changed as well. We have to assume things can and will break and we have to make sure it possible to understand what is happening in our applications behind the scenes at any given time.

## Observability 

The Wikipedia definition of observability is
> observability is a measure of how well internal states of a system can be inferred from knowledge of its external outputs. 

In software, these external outputs are usually grouped as *Logs*, *Metrics*, and *Request Traces*. Called *The Three Pillars of Observability*, they solve different problems, have different weaknesses and strengths. 

## Monitoring

Monitoring is another keyword that is used together with observability. It is the act of collecting these external outputs and after aggregating, analyzing and visualizing them, creating meaningful alerts with this data. These two concepts are closely related because the higher observability a system has, the better you can monitor it. 

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/ltr64lsw3tx2bo3cjomv.jpeg)

## Expected and Unexpected Problems 

Monitoring is helpful when expected things fail. And by expected, I mean things we can set alerts on beforehand. We can start monitoring common things every single software system has like CPU, memory or disk usage, request counts, 4xx responses, 5xx responses and average/max durations. Then we can set alarms on Exceptions from the codebase by checking logs or something more advanced. We can also take it one step further and monitor application/domain specific events like login requests or purchase event numbers, revenue etc. 

Observability on the other hand helps with dealing with unexpected problems as well. To find the root cause of a completely new problem we can follow steps like this:
1. check metrics to search for an anomaly(long durations, 5xx responses etc.)
1. focusing on a specific timeframe in logs, we can try to find a problematic event. We can search for the relevant log entries for the same request with a traceId
1. using the same traceId from logs, we can analyze the lifecycle of problematic requests

Sometimes we can find causes of problems very easily and go on with our lives and sometimes even with all the data in our hands, it is really difficult to find something concrete. There is always room for improvement with dashboards, alert runbooks and better log messages. 

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/gepjcdfzxgkpo77dzf8i.jpeg)

## Benefits

Both observability and monitoring are highly valuable concepts because in an ideal world:
1. If there are no problems in the system, we should be able to relax and sleep at night knowing that we would be notified with an alert if and when something goes wrong. 
1. And when there is a problem, we should first be notified and then have the required information available to investigate the root cause and do something in a short time. 
1. Another huge benefit of having an observable and monitored system is that it provides confidence and speed to developers. They can know if a change has caused any problems or not. And if it did, they are aware of the problem as soon as possible and more consistently.

## Future

With successful projects like OpenTelemetry(previously OpenTracing and OpenCensus) and OpenMetrics there is a lot of discussion and standardization taking place in this area. It is an exciting field with lots of things to learn so I hope to learn and write more about this.
