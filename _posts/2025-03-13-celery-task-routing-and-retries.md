---
title: "Celery Task Routing & Error Handling: Queues, Retries & Failures"
date: 2025-03-13
author: "Usman Asif"
description: "Learn how to route tasks to different queues, prioritize high vs. low-priority tasks, implement retries with exponential backoff, and handle errors using Celery’s Dead Letter Queues (DLQ)."
---


## 🚀 Introduction

Celery provides powerful features for routing tasks to specific queues and handling retries when tasks fail. Efficient task routing ensures that critical jobs run with priority, while a solid retry mechanism helps improve system resilience.

In this post, we'll cover:

-   Assigning tasks to different queues
    
-   Configuring `CELERY_ROUTES`, `task_queues`, and `task_routes`
    
-   Handling high-priority vs. low-priority tasks
    
-   Implementing retries with `autoretry_for` and `retry`
    
-   Using exponential backoff for retries
    
-   Handling failed tasks with Dead Letter Queues (DLQ)
    

----------

## 📌 Task Routing & Queues in Celery

By default, all tasks go into the `celery` queue. However, in production systems, you often need separate queues for different workloads.

### 1️⃣ Defining Task Queues

Celery allows you to define custom queues in your configuration:

{% highlight python %}
from kombu import Queue

CELERY_QUEUES = (
    Queue('high_priority'),
    Queue('low_priority'),
    Queue('background_tasks')
)
{% endhighlight %}

### 2️⃣ Routing Tasks to Queues

You can route tasks dynamically using `task_routes`:

{% highlight python %}
CELERY_TASK_ROUTES = {
    'tasks.process_order': {'queue': 'high_priority'},
    'tasks.send_email': {'queue': 'low_priority'},
    'tasks.generate_report': {'queue': 'background_tasks'},
}
{% endhighlight %}

### 3️⃣ Starting Workers for Specific Queues

To process only a specific queue, run:

{% highlight python %}
celery -A myproject worker -Q high_priority --loglevel=info
{% endhighlight %}

This ensures that the worker only processes high-priority tasks.

----------

## ⚠️ Celery Task Retries & Error Handling

Retries prevent temporary failures from causing permanent data loss. Celery allows automatic and manual retries.

### 1️⃣ Using `autoretry_for`

Instead of handling exceptions manually, use `autoretry_for`:

{% highlight python %}
from celery import Celery

app = Celery('my_project', broker='pyamqp://guest@localhost//')

@app.task(autoretry_for=(Exception,), retry_kwargs={'max_retries': 5, 'countdown': 10})
def fetch_data():
    raise Exception("Temporary failure")
{% endhighlight %}

This task retries **5 times**, waiting **10 seconds** between retries.

### 2️⃣ Using Exponential Backoff

To increase retry intervals dynamically:

{% highlight python %}
from celery.exceptions import Retry
import random

@app.task(bind=True, max_retries=5)
def process_data(self):
    try:
        # Simulate failure
        if random.choice([True, False]):
            raise ValueError("Something went wrong")
    except Exception as e:
        countdown = 2 ** self.request.retries  # Exponential backoff
        raise self.retry(exc=e, countdown=countdown)
{% endhighlight %}

### 3️⃣ Handling Failed Tasks with Dead Letter Queues (DLQ)

Failed tasks should go to a **Dead Letter Queue (DLQ)** for manual inspection.

Modify your Celery configuration to define a `dlq`:

{% highlight python %}
from kombu import Exchange, Queue

CELERY_TASK_QUEUES = (
    Queue('high_priority', exchange=Exchange('high', type='direct'), routing_key='high'),
    Queue('dlq', exchange=Exchange('dlx', type='direct'), routing_key='dlq'),
)
{% endhighlight %}

Then, configure Celery to send failed tasks to the DLQ:

{% highlight python %}
CELERY_TASK_DEFAULT_EXCHANGE_TYPE = 'direct'
CELERY_TASK_DEFAULT_QUEUE = 'high_priority'
CELERY_TASK_DEFAULT_ROUTING_KEY = 'high'
CELERY_TASK_REJECT_ON_WORKER_LOST = True
CELERY_TASK_RETRY_ON_FAILURE = False
{% endhighlight %}

----------

## 🔍 Monitoring & Debugging Celery Tasks

### 1️⃣ Enabling Celery Events

Use **Flower** to monitor Celery workers:

{% highlight python %}
pip install flower
celery -A myproject flower
{% endhighlight %}

Access the UI at http://localhost:5555 to see task status, failures, and retries.

### 2️⃣ Debugging Stuck Tasks

To check active tasks, use:

{% highlight python %}
celery -A myproject inspect active
{% endhighlight %}

For failed tasks:

{% highlight python %}
celery -A myproject inspect failed
{% endhighlight %}

----------

## 🎯 Conclusion

With task routing, retries, and error handling in place, your Celery setup is now more **resilient and efficient**.

✅ Use **task routes** to prioritize queues.  
✅ Implement **automatic retries** for transient failures.  
✅ Configure **Dead Letter Queues** for failed tasks.  
✅ Monitor workers with **Flower & Celery events**.

This ensures smooth task execution even under high load! 🚀