---
title: Jupyter 404 error - "Blocking Cross Origin API" - Azure HDInsight
description: Jupyter server 404 "Not Found" error due to "Blocking Cross Origin API" in Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 04/29/2022
---

# Scenario: Jupyter server 404 "Not Found" error due to "Blocking Cross Origin API" in Azure HDInsight

This article describes troubleshooting steps and possible resolutions for issues when using Apache Spark components in Azure HDInsight clusters.

## Issue

When you access the Jupyter service on HDInsight, you see an error box saying "Not Found". If you check the Jupyter logs, you will see something like this:

```log
[W 2018-08-21 17:43:33.352 NotebookApp] 404 PUT /api/contents/PySpark/notebook.ipynb (10.16.0.144) 4504.03ms referer=https://pnhr01hdi-corpdir.msappproxy.net/jupyter/notebooks/PySpark/notebook.ipynb
Blocking Cross Origin API request.  
Origin: https://xxx.xxx.xxx, Host: pnhr01.j101qxjrl4zebmhb0vmhg044xe.ax.internal.cloudapp.net:8001
```

You may also see an IP address in the "Origin" field in the Jupyter log.

## Cause

This error can be caused by a couple things:

- If you have configured Network Security Group (NSG) Rules to restricts access to the cluster. Restricting access with NSG rules will still allow you to directly access Apache Ambari and other services using the IP address rather than the cluster name. However, when accessing Jupyter, you could see a 404 "Not Found" error.

- If you have given your HDInsight gateway a customized DNS name other than the standard `xxx.azurehdinsight.net`.

## Resolution

1. Modify the jupyter.py files in these two places:

    ```bash
    /var/lib/ambari-server/resources/common-services/JUPYTER/1.0.0/package/scripts/jupyter.py
    /var/lib/ambari-agent/cache/common-services/JUPYTER/1.0.0/package/scripts/jupyter.py
    ```

1. Find the line that says: `NotebookApp.allow_origin='\"https://{2}.{3}\"'` And change it to: `NotebookApp.allow_origin='\"*\"'`.

1. Restart the Jupyter service from Ambari.

1. Typing `ps aux | grep jupyter` at the command prompt should show that it allows for any URL to connect to it.

This is a less secure than the setting we already had in place. But it is assumed access to the cluster is restricted and that one from outside is allowed to connect to the cluster as we have NSG in place.

## Next steps

[!INCLUDE [troubleshooting next steps](../includes/hdinsight-troubleshooting-next-steps.md)]
