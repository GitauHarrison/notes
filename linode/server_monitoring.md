# Monitor Your Server on Linode

Now that your Linode is up and running, it’s time to think about monitoring and maintaining your server. This tutorial discusses some essential tools and skills we can use to check our server's resources. In the process, we will learn how to monitor the availability and performance of our system, manage our logs and update the server's software.


Multiple things go into monitoring a server. For example, we might be interested in monitoring the following aspects of our server:

- [Availability of the server](#availability-of-the-server)
- [Performance of the server](#performance-of-the-server)

It is therefore important to first assess what needs we have before embarking on a server monitoring mission.

## Availability of the Server

Not everyone needs to monitor the availability of their server. If you are running a very basic application such as a morning quote website, you may not necessarily need to worry about service interruptions. Occassional inconviniences of the website going offline for a few minutes may not justify the time it takes to set up and configure an availability monitoring tool. 

However, if you depend on your website, say for livelihood, then it is a necessity to monitor your server. Once set up, the tool actively watches the server and services and alerts us when they are unavailable. We will be able to troubleshoot the problem and restore the service as soon as possible.

There are a handful of tools that we can use to monitor the availability of a server. 

- If we are running multiple servers, we can use [Elastic Stack](https://www.elastic.co/elastic-stack/). It  includes Elasticsearch, Logstash, and Kibana, is a troika of tools that provides a free and open-source solution that searches, collects and analyzes data from any source and in any format and visualizes it in real time. I will not go into the details of how to configure the server to use Elastic Stack for now.
- If we are running only a single server, we may consider using a third-party service to monitor our linode. 
- Linode offers [Linode Managed](https://www.linode.com/managed), an expert 24/7 monitoring service. It carries no obligation or contract and costs $100 per month, per Linode on your account.


### Configure Shutdown Watchdog

Occassionally, a linode may power off unexpectedly, making the server unavailable. In this case, Linode offers a shutdown watchdog called Lassie that automatically reboots a linode in such instances. It is not an unavailability monitoring tool, but it is useful in getting a linode back online.

Log in to your Linode to see available Linodes:
![Available linodes](/images/linode/server_monitoring/dashboard.png)

Notice that I have 2 linodes on my account, _official_personal_website_ and _tinkereducationnewsletter_. I will show you how to configure Lassie using the _official_personal_website_ linode.

I will click on it to access more data on the linode. What I am interested in is the "Settings" tab.
![Settings Tab](/images/linode/server_monitoring/settings.png)

Scroll to the bottom of the "Settings" tab to see the "Shutdown Watchdog" section. Toggle the key to enable this feature.
![Lassie](/images/linode/server_monitoring/lassie.png)


## Performance of the Server

For vital server and service performance metrics, performance monitoring tools are used. These tools can be equated to a car's dashboard which shows all car performance details such as speed and fuel consumption. We will begin by first looking at the default tools that monitor performance of a server then gradually check out a few more technical tools we can use.


### Linode Cloud Manager

Once our linode is up and running, Linode offers us the Cloud Manager in our dashboard with a few performance data. This data can be accessed by clicking on a linode, in my case the _official_personal_website_ linode.

![Linode analytics](/images/linode/server_monitoring/analytics.png)

The graph constains the following sections:

- CPU %: It shows how my linode's CPU is being utilized. 
- IPv4 network traffic: It keeps tabs with how much incoming and outgoing bandwidth the server is using.
- IPv6 network traffic: It checks how much bandwidth has been transferred to IPv6.
- Disk I/O: It checks the size of my Linode's disk, if it is full or not.

Chances are that you may not understand the graphs. It may be difficult to tell apart what numbers are normal and those that are abnormal.


### Email Alerts

Linode Cloud Manager allows us to configure email alerts that notify us when certain performance thresholds are reached.

![Email Alerts](/images/linode/server_monitoring/email_alerts.png)

In the illustration above, I have configured email notifications when the CPU Usage is 90% and above. To enable a particular threshold, toggle the appropriate switch, set a value and click the "Save" button to save the email alert threshold.

When we receive such an alert, it does not mean there is something wrong with the Linode. It simple means that the server is operating above a set threshold.


## Linux System Monitoring Fundamentals

Monitoring tools help to reassure us when things are working right, they help us to recognize odd behaviour, performance anomalies and their sources when the server misbehaves. All server monitoring tools have a few things in common. 

- They set a goal that ensures a server is performing optimally
- Provide administrative data
- Sometimes automate responses to anomalies

Data on each key performance indicator (KPI), network connectivity and application availability is collected and used for analysis. For example, working hardware, available server, server resources are sufficient, no bottlenecks are slowing things down and visualization of data.

Thankfully, we have dozens of server system monitoring tools built into Linux. I will show you how to use the `top` command to see avaiable Linux processes in CPU activity order. Understandably, there are a dozen more such as [System Activity Report (sar)](https://linux.die.net/man/1/sar), [Vmstat](https://linux.die.net/man/8/vmstat), [Monitorix](https://www.monitorix.org/), [Nethogs](https://github.com/raboof/nethogs), [Glance](https://nicolargo.github.io/glances/), [htop](https://htop.dev/) and [Netdata](https://www.netdata.cloud/).

 
### Monitor Server Performance using `top`

If we can see a server's processor activity in real-time, we are more likely to discover and diagonise any CPU and memory usage problems. The `top` command can assist with monitoring.

In your server's terminal, run the command below:

```python
$ top
```
![Top running](/images/linode/server_monitoring/top_running.png)

This screen contains a variety of information regarding the server.

```python
top - 14:56:17 up 127 days, 22:19,  2 users,  load average: 0.01, 0.01, 0.00
```

- The first line contains the **time, the uptime and load averages of the server**. The load average is displayed over 1, 5, and 15 minutes to provide a better overall look at the load my server has undertaken. 
- To properly read the load average, we need to know how many CPUs our Linode has. If there is 1 CPU, then a load average of 1.00 eans that the server is operating at its capacity. This number increases to 2 if the number of CPUs is 2, etc.
- A load average of 0.70 for a Linode with 1 core is generally considered a threshold. Anything higher requires reconfiguration of resources or the need to upgrade.


```python
Tasks: 118 total,   1 running, 117 sleeping,   0 stopped,   0 zombie
```

- The second line is a **list of tasks and their various states**.

```python
%Cpu(s):  0.3 us,  0.3 sy,  0.0 ni, 99.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
```

- The third line is the **CPU percentages**:
  - user CPU time (`us`)
  - System CPU time (`sy`)
  - Nice time (`ni`) - time spend on low prioity processes
  - Idle time (`id`)
  - Time spent on wait I/O processes (`wa`)
  - Time handling hardware interruptions (`hi`)
  - Time handling software interruptions (`si`)
  - Steal time (`st`) - time stolen from the virtual machine

- The forth line is the **server's memory usage** in kilobytes

```python
MiB Mem :    976.8 total,     97.0 free,    321.5 used,    558.3 buff/cache
```

- The fifth line is the **server's swap usage** in kilobytes

```python
MiB Swap:    512.0 total,    423.0 free,     89.0 used.    497.1 avail Mem
```

Thereafter, we have a heading with a list of processes and related data

![Top heading](/images/linode/server_monitoring/top_heading.png)

- **PID**: Process ID
- **USER**: The username of the task owner
- **PR**: The task priority from -20 - 19, with -20 being the most important
- **NI**: The _nice value_ which augments the priority of a task. Negative values increase a task's priority while postive values decrease it.
- **VIRT**: The virtual memory (both RAM and swap combined) used
- **RES**: The resident non-swapped, physical memory in kilobytes (usually)
- **SHR**: The shared memory size, or memory that could be allocated to other processes
- **S**: The process status. `R` for running, `D` for sleeping and unable to be interrupted, `S` sleeping and able to be interrupted, `T` for traced/stopped and `Z` for zombie
- **%CPU**: CPU percentage since the last `top` update
- **%MEM**: Memory (RAM) percentage since the last `top` update
- **TIME+**: Cumulative CPU time that the process and children processes have used
- **COMMAND**: Name of process


### `top` commands

The `top` command can be used in conjunction with other commands both on the commandline or interactively. Important commandline options include:

- `-d [interval]`: Sets the delay time that `tip` uses to refresh results
- `-i`: Toggles whether or not the idle processes are shown
- `-p [PID]`: Allows the user to filter `top` so only defined processes are shown
- `-u [username]`: Filters by user
- `-n [limit]`: Sets `top` to run for a set amount of intervals before exiting
- `b`: Runs `top` in batch mode, which is ideal for log files and in conjunction with other programs

```python
$ top -b -p3304014 -d10 -n2

# Output

top - 15:53:58 up 127 days, 23:16,  2 users,  load average: 0.09, 0.02, 0.01
Tasks:   1 total,   0 running,   1 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :    976.8 total,     93.7 free,    321.3 used,    561.8 buff/cache
MiB Swap:    512.0 total,    423.0 free,     89.0 used.    497.0 avail Mem 

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
3304014 gitauha+  20   0   90192  59052   7200 S   0.0   5.9   0:08.62 gunicorn

top - 15:54:03 up 127 days, 23:16,  2 users,  load average: 0.08, 0.02, 0.00
Tasks:   1 total,   0 running,   1 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.2 us,  0.0 sy,  0.0 ni, 99.8 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :    976.8 total,     93.7 free,    321.3 used,    561.8 buff/cache
MiB Swap:    512.0 total,    423.0 free,     89.0 used.    497.0 avail Mem 

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
3304014 gitauha+  20   0   90192  59052   7200 S   0.0   5.9   0:08.62 gunicorn

```

The above `top` command logs the process identified by the PID `3304014` after a delay of 10 seconds in 2 cycles.

Interactively, we can issue the following commands in an active `top` session:

- `return` or `space`: Instantly update the screen
- `d` or `s`: Alter the delay time
- `H`: Show individual threads for all processes
- `i`: Toggles whether idle processes will be displayed
- `U` or `u`: Filter the process by the owner's username
- `k`: Kill a process. You will be prompted to enter the PID
- `q`: Quit


### Commands Similar to `top`

There is [htop](http://hisham.hm/htop/), which is similar to `top`, but offers an easier interface with color, mouse operations, and horizontal and vertical scrolling, making it more intuitive.

To use it, we first need to install it by running th command:

```python
$ sudo apt install htop
```

Running is similar to `top`:

```python
$ htop
```

![htop](/images/linode/server_monitoring/htop.png)

You can use your mouse to scroll the interactive process viewer. You can click on a process using yoru mouse to highlight it then press `k`, for example, to kill it. At the bottom, you will notice a few buttons that you can click on.