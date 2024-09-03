---
title: "Powershell Jobs"
categories:
  - Powershell
  - Powershell Jobs
tags:
  - PowerShell
  - Powershell Jobs
  - Asynchronous Powershell
---


## First post – First exercise – Working with Powershell Jobs

Alright, full disclosure time. I launched PowerShell Gym around six months back with every intention to hit you with a weekly post. Fast forward to now, and well, let’s just say the initial post was more of a one-hit wonder. Why the radio silence? I could spin a few excuses, but honestly, there aren’t any good ones. Yet here we are, on this fine Friday night of August 30th, and something just clicked. Why not breathe some life back into this blog? So, that’s exactly what we’re doing.

Welcome (back) to the first post and the first exercise. Even though I haven’t been sharing, believe me, I’ve been knee-deep in PowerShell scripts and projects all this while. Time to get those experiences out of my system and onto this blog. Let’s get this PowerShell party restarted!

## Powershell Jobs

Okay, first, let’s dive into what PowerShell jobs are, and then we’ll create an exercise for ourselves. As you might know, PowerShell typically works by doing one job after the other (also called: synchronously). But what if we want something different, something asynchronous? And yeah, I might have mentioned this before, but I want this blog to be as practical as possible, so why the hell would you want something to run asynchronously?

Running tasks asynchronously in PowerShell can improve the efficiency and responsiveness of your scripts. This approach is especially useful when handling time-consuming operations that don’t need to be completed one after the other (=sequentially). For example, if you’re monitoring remote servers using a Telegraf or Nagios agent (for monitoring purposes), and you want to make sure the service is running, you can initiate all requests at once. This reduces the total time spent waiting for tasks to complete one after another, thereby speeding up the process and saving more time for other things!

*Wait, why doing this?*

Your text captures the essence of why asynchronous tasks can be more efficient compared to using a foreach loop for certain operations. Here’s a slightly refined version of your text to enhance clarity and flow:

You might be thinking, “Why not just use a foreach loop?” Here’s the thing: a foreach loop checks each server one at a time (remember, that’s synchronously). So if one server takes forever to respond, you’re stuck waiting on that one before moving to the next. It’s like standing in a slow-moving line.

Okay, lets get scripting!

we’re about to write a PowerShell script to check if the Telegraf agent is running on multiple remote servers, and we’ll do this asynchronously.

````powershell
# all your precious servers
$serverNames = @("Server01", "Server02", "Server03")

# the service we want to check
$serviceName = "telegraf"

foreach ($Server in $serverNames) {
    Start-Job -ScriptBlock {
        param($Server, $serviceName)

        # connection to your server (assuming you got the right permissions)
        $session = New-PSSession -ComputerName $Server

        # starting the service on the remote server
        start-Service -Name $serviceName -Session $session

        # Closing the session
        Remove-PSSession -Session $session

    } -ArgumentList $Server, $serviceName
}

# check the status of all jobs
Get-Job | Receive-Job
````

The Get-Job &#124; Receive-Job line in the script plays a crucial role at the end. After starting asynchronous jobs that check and start the Telegraf service on our multiple precious servers, this command fetches the results of those jobs. Get-Job lists all the background jobs running or completed, and Receive-Job retrieves the output from these jobs. This way, you can see the status of the service on each server, ensuring that each action has been executed and completed successfully. This helps in monitoring and verifying that the Telegraf service is up and running across your server network.
