# Troubleshooting the CloudWatch Agent<a name="troubleshooting-CloudWatch-Agent"></a>

Use the following information to help troubleshoot problems with the CloudWatch agent\. 

**Topics**
+ [Installing the CloudWatch Agent Using Run Command Fails](#CloudWatch-Agent-installation-fails)
+ [The CloudWatch Agent Won't Start](#CloudWatch-Agent-troubleshooting-cannot-start)
+ [Verify That the CloudWatch Agent is Running](#CloudWatch-Agent-troubleshooting-verify-running)
+ [Where Are the Metrics?](#CloudWatch-Agent-troubleshooting-no-metrics)
+ [Agent won't start and error mentions Amazon EC2 region](#CloudWatch-Agent-troubleshooting-EC2-region)
+ [CloudWatch Agent Files and Locations](#CloudWatch-Agent-files-and-locations)
+ [Logs Generated by the CloudWatch Agent](#CloudWatch-Agent-troubleshooting-loginfo)
+ [Stopping and Restarting the CloudWatch Agent](#CloudWatch-Agent-troubleshooting-stopping-restarting)
+ [Starting the CloudWatch Agent at System Startup](#CloudWatch-Agent-troubleshooting-starting-system-startup)

## Installing the CloudWatch Agent Using Run Command Fails<a name="CloudWatch-Agent-installation-fails"></a>

To install the CloudWatch agent using Systems Manager Run Command, the SSM Agent on the target server must be version 2\.2\.93\.0 or later\. If your SSM Agent is not the correct version, you may see errors that include the following messages:

```
no latest version found for package AmazonCloudWatchAgent on platform linux
```

```
failed to download installation package reliably
```

For information about updating your SSM Agent version, see [Installing and Configuring SSM Agent](http://docs.aws.amazon.com/systems-manager/latest/userguide/ssm-agent.html)\.

## The CloudWatch Agent Won't Start<a name="CloudWatch-Agent-troubleshooting-cannot-start"></a>

If the CloudWatch agent fails to start, there might be an issue in your configuration\. Configuration information is logged in the **configuration\-validation\.log** file\. This file is located in `/opt/aws/amazon-cloudwatch-agent/logs/configuration-validation.log` on Linux servers, and in `$Env:ProgramData\Amazon\AmazonCloudWatchAgent\Logs\configuration-validation.log` on servers running Windows Server\.

## Verify That the CloudWatch Agent is Running<a name="CloudWatch-Agent-troubleshooting-verify-running"></a>

You can query the CloudWatch agent to find whether it is running or stopped\.

You can use AWS Systems Manager to do this remotely\. You can also use the command line, but only to check the local server\.

**To query the status of the CloudWatch agent using Run Command**

1. Open the Systems Manager console at [https://console\.aws\.amazon\.com/systems\-manager/](https://console.aws.amazon.com/systems-manager/)\.

1. In the navigation pane, choose **Run Command**\.

   \-or\-

   If the AWS Systems Manager home page opens, scroll down and choose **Explore Run Command**\.

1. Choose **Run command**\.

1. In the **Command document** list, choose **AmazonCloudWatch\-ManageAgent**\.

1. In the **Target** area, choose the instance to check\.

1. In the **Action** list, choose **status**\.

1. Leave **Optional Configuration Source** and **Optional Configuration Location** blank\.

1. Choose **Run**\.

If the agent is running, the output resembles the following:

```
{
       "status": "running",
       "starttime": "2017-12-12T18:41:18",
       "version": "1.73.4"
}
```

If the agent is stopped the `"status"` field displays `"stopped"`\.

**To query the status of the CloudWatch agent locally using the command line**
+ On a Linux server, type the following:

  ```
  sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status
  ```

  On a server running Windows Server, type the following in PowerShell as an administrator:

  ```
  & $Env:ProgramFiles\Amazon\AmazonCloudWatchAgent\amazon-cloudwatch-agent-ctl.ps1 -m ec2 -a status
  ```

## Where Are the Metrics?<a name="CloudWatch-Agent-troubleshooting-no-metrics"></a>

If the CloudWatch agent has been running but you cannot find metrics collected by it in the AWS Management Console or the AWS CLI, confirm that you are using the correct namespace\. By default the namespace for metrics collected by the agent is `CWAgent`\. You can customize this namespace using the **namespace** field in the **metrics** section of the agent configuration file\. If you do not see the metrics that you expect, check the configuration file to confirm the namespace being used\.

When you first download the CloudWatch agent package, the agent configuration file is `amazon-cloudwatch-agent.json`\. This file is located in the directory where you ran the configuration wizard, or you may have moved it to a different directory\. If you use the configuration wizard, the agent configuration file output from the wizard is named `config.json`\. For more information about the configuration file, including the **namespace** field, see [ CloudWatch Agent Configuration File: Metrics Section](CloudWatch-Agent-Configuration-File-Details.md#CloudWatch-Agent-Configuration-File-Metricssection)\. 

## Agent won't start and error mentions Amazon EC2 region<a name="CloudWatch-Agent-troubleshooting-EC2-region"></a>

If the agent will not start and the error message mentions an Amazon EC2 region endpoint, you may have configured the agent to need access to the Amazon EC2 endpoint but not granted that access\.

For example, if you specify a value for the `append_dimensions` parameter in the agent configuration file that depends on Amazon EC2 metadata, and you use proxies, you must make sure that the server can access the endpoint for Amazon EC2\. For more information about these endpoints, see [Amazon Elastic Compute Cloud \(Amazon EC2\)](http://docs.aws.amazon.com/general/latest/gr/rande.html#ec2_region) in the *Amazon Web Services General Reference*\.

## CloudWatch Agent Files and Locations<a name="CloudWatch-Agent-files-and-locations"></a>

The following table lists the files installed by and used with the CloudWatch agent, along with their locations on servers running Linux or Windows Server\.


| File | Linux Location | Windows Server Location | 
| --- | --- | --- | 
|  The control script that controls starting, stopping, and restarting the agent\. |  /opt/aws/amazon\-cloudwatch\-agent/bin/amazon\-cloudwatch\-agent\-ctl  |  $Env:ProgramFiles\\Amazon\\AmazonCloudWatchAgent\\amazon\-cloudwatch\-agent\-ctl\.ps1  | 
|  The log file the agent writes to\. You may need to attach this when contacting customer support\. |  /opt/aws/amazon\-cloudwatch\-agent/logs/amazon\-cloudwatch\-agent\.log   |  $Env:ProgramData\\Amazon\\AmazonCloudWatchAgent\\Logs\\amazon\-cloudwatch\-agent\.log  | 
|  Agent configuration validation file\. |  /opt/aws/amazon\-cloudwatch\-agent/logs/configuration\-validation\.log  |  $Env:ProgramData\\Amazon\\AmazonCloudWatchAgent\\Logs\\configuration\-validation\.log  | 
|  The JSON file used to configure the agent, immediately after the wizard creates it\. For more information, see [Create the CloudWatch Agent Configuration File](create-cloudwatch-agent-configuration-file.md)\. |  /opt/aws/amazon\-cloudwatch\-agent/bin/config\.json   |  $Env:ProgramData\\Amazon\\AmazonCloudWatchAgent\\config\.json  | 
|  The JSON file used to configure the agent, if this configuration file has been downloaded from Parameter Store\. |  /opt/aws/amazon\-cloudwatch\-agent/etc/amazon\-cloudwatch\-agent\.json  |  $Env:ProgramData\\Amazon\\AmazonCloudWatchAgent\\amazon\-cloudwatch\-agent\.json  | 
|  TOML file used to specify region and credential information to be used by the agent, overriding system defaults\. |  /opt/aws/amazon\-cloudwatch\-agent/etc/common\-config\.toml  |  $Env:ProgramData\\Amazon\\AmazonCloudWatchAgent\\common\-config\.toml  | 
|  TOML file generated from the JSON configuration file when the CloudWatch agent is started\. |  /opt/aws/amazon\-cloudwatch\-agent/etc/amazon\-cloudwatch\-agent\.toml   |  $Env:ProgramData\\Amazon\\AmazonCloudWatchAgent\\amazon\-cloudwatch\-agent\.toml  | 

## Logs Generated by the CloudWatch Agent<a name="CloudWatch-Agent-troubleshooting-loginfo"></a>

The agent generates a log while it runs\. This log includes troubleshooting information\. This log is the **amazon\-cloudwatch\-agent\.log** file\. This file is located in `/opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log` on Linux servers, and in `$Env:ProgramData\Amazon\AmazonCloudWatchAgent\Logs\amazon-cloudwatch-agent.log` on servers running Windows Server\.

You can configure the agent to log additional details in the **amazon\-cloudwatch\-agent\.log** file\. In the agent configuration file, in the **agent** section, set the **debug** field to **true**, then reconfigure and restart the CloudWatch agent\. To disable the logging of this extra information, set the **debug** field to **false** reconfigure and restart the agent\. For more information, see [ Manually Create or Edit the CloudWatch Agent Configuration File](CloudWatch-Agent-Configuration-File-Details.md)\.

## Stopping and Restarting the CloudWatch Agent<a name="CloudWatch-Agent-troubleshooting-stopping-restarting"></a>

You can manually stop the CloudWatch agent using either AWS Systems Manager or the command line\. When you stop it manually, you also prevent it from automatically starting at system reboot\.

**To stop the CloudWatch agent using Run Command**

1. Open the Systems Manager console at [https://console\.aws\.amazon\.com/systems\-manager/](https://console.aws.amazon.com/systems-manager/)\.

1. In the navigation pane, choose **Run Command**\.

   \-or\-

   If the AWS Systems Manager home page opens, scroll down and choose **Explore Run Command**\.

1. Choose **Run command**\.

1. In the **Command document** list, choose **AmazonCloudWatch\-ManageAgent**\.

1. In the **Targets** area, choose the instance where you installed the CloudWatch agent\.

1. In the **Action** list, choose **stop**\.

1. Leave **Optional Configuration Source** and **Optional Configuration Location** blank\.

1. Choose **Run**\.

**To stop the CloudWatch agent locally using the command line**
+ On a Linux server, type the following:

  ```
  sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a stop
  ```

  On a server running Windows Server, type the following in PowerShell as an administrator:

  ```
  & $Env:ProgramFiles\Amazon\AmazonCloudWatchAgent\amazon-cloudwatch-agent-ctl.ps1 -m ec2 -a stop
  ```

To restart the agent, follow the instructions in [Start the CloudWatch Agent](install-CloudWatch-Agent-on-first-instance.md#start-CloudWatch-Agent-EC2)\.

## Starting the CloudWatch Agent at System Startup<a name="CloudWatch-Agent-troubleshooting-starting-system-startup"></a>

On both Linux and Windows Server, the CloudWatch agent installation scripts register the agent as a system service\. This service is configured to start when your instance or server boots, if the agent was running before the reboot occurred\. The CloudWatch agent relies on having been previously configured for it to properly start\. If you are preparing an AMI or machine image that starts the CloudWatch agent on newly launched Amazon EC2 instances or on\-premises servers, including instances launched by Auto Scaling and Spot Fleet, ensure that the `amazon-cloudwatch-agent.toml` and `common-config.toml` files are included in your AMI\. These files are located in `/opt/aws/amazon-cloudwatch-agent/etc/` on Linux servers, and in `$Env:ProgramData\Amazon\AmazonCloudWatchAgent\` on servers running Windows Server\. 

If your instance starts and these files are not present, the CloudWatch agent does not start correctly\.