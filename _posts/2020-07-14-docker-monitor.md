---
layout: post
title: "Docker Monitor"
date: 2020-07-14 15:23:49 +0100
categories: docker
---
For professional reasons I had to start using Docker and the experience has been far from perfect. Among the many problems I had to deal with was to recurrently have the application I'm working on begin to misbehave, only to trace the cause back to an unexpectedly exited Docker container. There might be a better solution to this, but the quick fix was a quick and dirty PowerShell script:

```powershell
While (1)
{
    $containers = & docker ps -a

    ForEach ($container in $($containers -split "`r`n"))
    {
        If ($container -match "Exited")
        {
            Get-Date
            Write-Host $container
            Write-Host "Container exited; restarting"
            $containerId = $container.Substring(0, 12)

            # Restart the container
            docker start $containerId
        }
    }
    Start-Sleep -Seconds 60
}
```

Every minute a check will be performed on whether any container exited, and all caught ones will be promptly restarted.
