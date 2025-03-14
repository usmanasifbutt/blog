---
title: "Scheduling Tasks with Celery Beat: Periodic Tasks & Crontab"
date: 2025-03-13
author: "Usman Asif"
description: "Learn how to use Celery Beat for scheduling periodic tasks, setting up interval and crontab schedules, and ensuring reliable execution of background jobs."
---


## 🚀 Introduction

Celery Beat is an add-on scheduler for Celery that enables **periodic task execution**. It acts as a lightweight cron replacement, allowing you to schedule tasks at fixed intervals or specific times using **IntervalSchedule** and **CrontabSchedule**.

This guide will walk you through setting up Celery Beat, defining periodic tasks, and understanding how Celery picks tasks from Beat.

----------

## 📌 What is Celery Beat & When to Use It?

Celery Beat is essential when you need tasks to run at scheduled intervals, such as:

-   **Database Cleanup**: Removing stale records every night.
    
-   **Email Reminders**: Sending notifications on a recurring basis.
    
-   **Data Synchronization**: Fetching data from APIs at regular intervals.
    
-   **Report Generation**: Generating reports at a specific time.
    

Unlike traditional **Celery workers** that process tasks **on demand**, Celery Beat ensures tasks run automatically based on a schedule.

----------

## 🛠️ Setting Up Celery Beat

### 1️⃣ Install Celery Beat

Install Celery Beat using pip:

{% highlight bash %} pip install django-celery-beat {% endhighlight %}

If you are using **Django**, add it to `INSTALLED_APPS` and apply migrations:

{% highlight python %} INSTALLED_APPS = [ 'django_celery_beat', ] {% endhighlight %}

Run migrations to create necessary tables:

{% highlight bash %} python manage.py migrate django_celery_beat {% endhighlight %}

### 2️⃣ Configure Celery to Use Beat

Ensure your `celery.py` file is correctly configured:

{% highlight python %} from celery import Celery

app = Celery('my_project', broker='pyamqp://guest@localhost//') app.config_from_object('django.conf:settings', namespace='CELERY')

# Auto-discover tasks from installed apps

app.autodiscover_tasks() {% endhighlight %}

Now, start the **Celery Beat scheduler**:

{% highlight bash %} celery -A my_project beat --loglevel=info {% endhighlight %}

This starts the periodic task scheduler that will send tasks to Celery workers based on defined schedules.

----------

## ⏳ Defining Periodic Tasks

Celery Beat provides two primary ways to schedule periodic tasks:

### 1️⃣ Using `IntervalSchedule` (Every X Seconds/Minutes/Hours)

Interval schedules allow tasks to run at fixed time intervals.

#### Example: Run a Task Every 10 Minutes

{% highlight python %} from celery.schedules import schedule from my_project.celery import app

@app.task def send_report(): print("Sending daily report...") {% endhighlight %}

Register this schedule in **Django Admin** under `Periodic Tasks`.

----------

### 2️⃣ Using `CrontabSchedule` (Specific Time Scheduling)

Crontab schedules allow you to run tasks at specific times, similar to a **Linux cron job**.

#### Example: Run a Task Every Day at Midnight

{% highlight python %} from celery.schedules import crontab from my_project.celery import app

@app.task def database_cleanup(): print("Cleaning up old records...") {% endhighlight %}

Define a **CrontabSchedule** for midnight execution:

{% highlight python %} from django_celery_beat.models import PeriodicTask, CrontabSchedule import json

schedule, _ = CrontabSchedule.objects.get_or_create(hour=0, minute=0)

PeriodicTask.objects.create( crontab=schedule, name="Database Cleanup Task", task="my_project.tasks.database_cleanup", args=json.dumps([]), ) {% endhighlight %}

This will execute `database_cleanup` every day at midnight.

----------

## 🔄 How Celery Picks Tasks from Beat

1.  **Celery Beat stores schedules in the database** (if using `django-celery-beat`).
    
2.  **Celery Beat periodically scans schedules** and publishes tasks to the queue.
    
3.  **Celery workers listen to the queue** and execute tasks when they are scheduled.
    

----------

## 🚨 Troubleshooting Celery Beat Issues

### 1️⃣ Tasks Are Not Running

-   Ensure Beat is running: `{% highlight bash %}celery -A my_project beat --loglevel=info{% endhighlight %}`
    
-   Ensure workers are running: `{% highlight bash %}celery -A my_project worker --loglevel=info{% endhighlight %}`
    
-   Check the `Periodic Tasks` table in Django Admin.
    

### 2️⃣ Duplicate Task Execution

-   Use **PeriodicTask.unique_id** to avoid multiple registrations.
    

### 3️⃣ Logs Are Not Showing Execution

-   Add logging to tasks:
    

{% highlight python %} import logging logger = logging.getLogger(**name**)

@app.task def my_task(): logger.info("Task executed successfully") {% endhighlight %}

----------

## 🎯 Conclusion

Celery Beat is a powerful tool for scheduling periodic tasks in a Celery-based system. By leveraging **IntervalSchedule** and **CrontabSchedule**, you can automate background jobs efficiently.

✅ Set up Celery Beat. ✅ Use **IntervalSchedule** for fixed intervals. ✅ Use **CrontabSchedule** for precise scheduling. ✅ Ensure workers are running to process scheduled tasks.

With Celery Beat, you can **automate repetitive background tasks** effortlessly! 🚀