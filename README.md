# Freemius Server Resources Monitoring (with Email Alerts)

This bash script is used for [Freemius](https://freemius.com)' servers monitoring. It monitors a Linux server CPU(s), RAM, and Disk(s) usage. When a resource reaches a specified threshold, it will automatically send an incident alert to the desired email address. Once the resource consumption goes under the threshold, the incident will be closed, and another alert will be sent to the address letting the sys-admin knowing that the incident is over.

The script supports unlimited levels of severity limits. So for example, you can have a `warning` severity limit as well as `critical` alerts.

## System Requirements
This bash script created for CentOS 6.X. It should also work smoothly with RHEL and Amazon Linux.
If you want to use it on a Debian or Ubuntu, you'll need to replace all the `yum` calls with `apt-get`. The rest should work.

## Motivation
We've been using [NewRelic](https://newrelic.com) for years and loved the product. Recently, they made changes in their plans and the cheapest plan that supports email alerts, which was one of the main features we've been using, starts from $300 per month. Paying $3,600 per year for email alerts doesn't make sense for us, so we decided to build this script for our internal use. Later, we decided to contribute it to the open-source community - saving others the several days of development. So enjoy!

## Usage
`bash server-monitoring.sh [options]`

|         Argument | Description                                                                |          |
|-----------------:|----------------------------------------------------------------------------|----------|
|  `--debug`, `-d` | If set to `true` will echo the progress.                                   | Optional |
|   `--info`, `-i` | If set to `true` just output the server's CPU, RAM, and disks consumption. | Optional |
|    `--from`,`-f` | The email address where the incident alerts are going to be sent from.     | Required |
|     `--to`, `-t` | The email address where the incident alerts are going to be sent to.       | Required |
|    `--cpu`, `-c` | The avg. CPU consumption incident severity limits.                         | Required |
| `--memory`, `-m` | The RAM consumption incident severity limits.                              | Required |
|   `--disk`, `-d` | The disk(s) consumption incident severity limits.                          | Required |

### Example
`bash server-monitoring.sh --debug=true --from=server@yourdomain.com --to=admin@yourdomain.com --cpu=warning=20:critical=50 --memory=warning=30:critical=60 --disk=warning=40:critical=60:fatal=70`

### Instructions
1. Select an email address that will be the source for the server incident alerts. Something like `server@yourdomain.com`.
2. The script is using `sendmail`'s most basic functionality. 99% that those emails will go directly to your spam. Thus, whitelist all messages from the selected address.
3. Copy the script to your server. Since the script is going to generate files for open incidents, we recommend to create a new folder named `server-monitoring` and copy the script into that folder.
4. To choose your resource consumption limits, first, execute the script in an `info` mode to learn your current server's resource consumption:

    `bash server-monitoring.sh --info=true`

    The output of the script will look like that:
    ```
    CPU:
    14
    MEMORY:
    26
    DISK:
    /dev/xvda
    34
    /dev/xvda1
    12
    ```
    The numbers represent the current resource consumption in percentage. For example, the disk `/dev/xvda` is 34% used.
    
    If you're getting `Command Not Found` error(s), try executing `dos2unix server-monitoring.sh` to convert the line endings to a Unix format.
5. After you know your resources normal state usage, come up with limits that you'd like to be alerted for.

`bash server-monitoring.sh --debug=true --from=server@yourdomain.com --to=admin@yourdomain.com --cpu=warning=20:critical=50 --memory=warning=30:critical=60 --disk=warning=40:critical=60:fatal=70`

### Explanation
- When the server's avg. CPU(s) consumption will increase above 20%; a `warning` CPU incident will be opened. If the CPU consumption increases beyond 50%, a `critical` CPU incident will be opened.
- When the server's memory/RAM consumption increases above 30%, a `warning` memory incident will be opened. If the RAM consumption increases beyond 60%, a `critical` memory incident will be opened.
- When any mounted disk usage increases above 40%, a `warning` disk incident will be opened. If a disk usage increases above the 60% threshold, a `critical` disk incident is opened. And when more than 70% of a disk is utilized, it will trigger a `fatal` incident.

## Ongoing Monitoring with a Cron Job
To set an ongoing monitoring:
1. Open the crontab file: `crontab -e`
2. Add the following line to the end of the file:

   `*/2 * * * * bash /server-monitoring/server-monitoring.sh --from=server@freemius.com --to=admin@freemius.com --cpu=warning=20:critical=50 --memory=warning=30:critical=60 --disk=warning=40:critical=60:fatal=70`
3. Type `:x` to save and exit.
