# Background Task Scheduler

This document provides a basic overview for a 'scheduler' which will be
  implemented as a background task to manage all the Report Schedules.

- [Other Options](#other-options)

## Overview

The background task manager will need to be able to do the following:

- Asynchronously pull in data from the ReportScheduling database on a regular basis
- Be able to analyze that data to see if that particular schedule needs to be
  run that day

## From Stackoverflow Article

Here is a link to the article: <https://stackoverflow.blog/2008/07/18/easy-background-tasks-in-aspnet/>
Here's an overview:

1. At startup, add an item to the `HttpRuntime.Cache` with a fixed expiration.
2. When the cache item expires, do your work such as `WebRequest`.
3. Re-add the item to the cache with a fixed expiration.

Here's the overview code:

```csharp
private static CacheItemRemovedCallback OnCacheRemove = null;

protected void Application_Start(object sender, EventArgs e)
{
    AddTask("DoStuff", 60);
}

private void AddTask(string name, int seconds)
{
    OnCacheRemove = new CacheItemRemovedCallback(CacheItemRemoved);
    HttpRuntime.Cache.Insert(name, seconds, null,
      DateTime.Now.AddSeconds(seconds), Cache.NoSlidingExpiration,
      CacheItemPriority.NotRemovable, OnCacheRemove);
}

public void CacheItemRemoved(string k, object v, CacheItemRemovedReason r)
{
    // Do stuff here if it matches our taskname, like WebRequest
    // Re-add our task so it recurs
    AddTask(k, Convert.ToInt32(v));
}
```

## Other Options

Here is a list of a few other options we can utilize for implementing the
  background tasks other than using the built-in caching:
1. [Quartz.NET](https://www.quartz-scheduler.net/)
    - Quartz.NET is an OpenSource project aimed at creating a free-for-commercial
      use Job Scheduler, with 'enterprise' features

2. [Windows Task Scheduler](https://msdn.microsoft.com/en-us/library/windows/desktop/aa383614(v=vs.85))
    - The Task Scheduler enables you to automatically perform routine tasks on a
      chosen computer
    - I'm not entirely sure how this will work, but it could be worth looking into.