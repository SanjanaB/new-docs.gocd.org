---
description: When run of of space, GoCD stops scheduling new pipelines by compressing large files, attaching a larger hard drive, or by deleting unused artifacts. 
keywords: Disk space, schedule pipelines, 
title: Running out of Disk Space
---


# Running out of disk space

After you've had GoCD running for a while, you may notice the following warning box when browsing GoCD:

![](../../images/1_low_disk_space_on_artifacts.png)

If you don't do anything about it, you'll end up seeing the following error:

![](../../images/2_out_of_disk_space_on_artifacts.png)

GoCD will stop scheduling new pipelines until you make more room, either by compressing large files, attaching a larger hard drive, or by deleting unused artifacts. You could also let GoCD manage artifact disk space by enabling auto purge of old artifacts.

## Auto delete artifacts

### Introduction

GoCD can be configured to automatically delete artifacts if the available disk space on the server is low. GoCD will purge artifacts when available disk space is lower than the given value. Artifacts will be purged upto the point when available disk space is greater than a defined value.

### Configuration

#### Specify artifact purge start and end limits

> You must be logged in as an admin user to configure this step.

1.  Navigate to the Admin section on the GoCD dashboard.
2.  Navigate to the Pipeline Management sub-section
3.  Specify when GoCD should begin to purge artifacts in the first edit box.
4.  Specify when GoCD should stop purging artifacts in the second edit box.

![Purge artifacts](../../images/pipeline_management.png)

#### Never delete artifacts for a stage

> You must be logged in as an admin user to configure this step.

You can disallow deletion of artifacts from a particular stage so that those artifacts are excluded during deletion. This option can be set in the stage editor for a pipeline. This option can be set for stages that are important so that artifacts for the stage are preserved.

1. Navigate to the admin section on the GoCD dashboard.
2. Navigate to the pipelines section and choose a pipeline to edit
3. Navigate to the stage settings for the stage

![Disable artifact cleanup](../../images/artifact_disable_stage.png)

4.Check the box 'Never Cleanup Artifacts'


### Also see...

-   [Managing artifacts and reports](../configuration/managing_artifacts_and_reports.html)
-   [Clean up after cancelling a task](../../advanced_usage/dev_clean_up_when_cancel.html)

## Compress large log files

In many cases, the easiest thing to do is compress some of the larger artifacts that you won't frequently have need for. For example, if you have a large log file named 'test.log' and you're running GoCD server on a unix machine, the following script will gzip those files that haven't been modified in the last 10 days

```
find /var/lib/go-server/logs/pipelines -name test.log -mtime +10 -type f -exec gzip -v '{}' \;
```

Now, if you add this to a system [crontab](http://en.wikipedia.org/wiki/Cron), your server can compress large artifacts automatically.

## Move the artifact repository to a new (larger) drive

If compressing large artifacts is not giving you enough free space, another thing you can do is attach a larger disk drive to store artifacts. After the drive is attached to the system, we can easily change the location GoCD uses for it's artifact repository.

-   Find the location of the GoCD configuration file
-   Navigate to the [Admin](../navigation/administration_page.html) section
![](../../images/topnav_admin.png)
-   Click on the "Config XML" tab
-   The location of the configuration file is listed here
![](../../images/4_find_config_location.png)
-   Install the new drive
-   Shut down GoCD server
-   Copy all files from the original artifact repository location to the new drive
-   Change the artifact repository location in the configuration file
``` 
<server artifactsDir="/path/to/new/artifacts">
    ...
</server>
```
-   Start up GoCD server and verify you still have access to old artifacts

## Delete unused artifacts

Another option for making more room is to remove unused (or easily recreatable) artifacts. You may also have old pipelines that you no longer need.

The directory structure of the artifact repository makes selecting which artifacts are safe to delete easier. The format is:
```
[artifacts-dir]/pipelines/[pipelineName]/[pipelineLabel]/[stageName]/[stageCounter]/[jobName]
```
> Keep in mind that there are two files that GoCD needs in order to display the [Job](../navigation/job_details_page.html) or [Stage](../navigation/stage_details_page.html) details pages

>-   cruise-output/console.log
>-   cruise-output/log.xml
