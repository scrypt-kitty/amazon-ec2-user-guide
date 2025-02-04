# Amazon EC2 Mac instances<a name="ec2-mac-instances"></a>

Amazon EC2 Mac instances natively support the macOS operating system\.
+ EC2 x86 Mac instances \(`mac1.metal`\) are built on 2018 Mac mini hardware powered by 3\.2 GHz Intel eighth\-generation \(Coffee Lake\) Core i7 processors\. 
+ EC2 M1 Mac instances \(`mac2.metal`\) are built on 2020 Mac mini hardware powered by Apple Silicon M1 processors\. 

EC2 Mac instances are ideal for developing, building, testing, and signing applications for Apple devices, such as iPhone, iPad, iPod, Mac, Apple Watch, and Apple TV\. You can connect to your Mac instance using SSH or Apple Remote Desktop \(ARD\)\.

**Note**  
The **unit of billing** is the **dedicated host**\. The instances running on that host have no additional charge\.

For more information, see [Amazon EC2 Mac Instances](https://aws.amazon.com/mac) and [Pricing](https://aws.amazon.com/mac/#Pricing)\.

**Topics**
+ [Considerations](#mac-instance-considerations)
+ [Instance readiness](#mac-instance-readiness)
+ [Launch a Mac instance](#mac-instance-launch)
+ [Connect to your Mac instance](#connect-to-mac-instance)
+ [Modify macOS screen resolution on Mac instances](#mac-screen-resolution)
+ [EC2 macOS AMIs](#ec2-macos-images)
+ [Update the operating system and software](#mac-instance-updates)
+ [EC2 macOS Init](#ec2-macos-init)
+ [EC2 System Monitoring for macOS](#mac-instance-system-monitor)
+ [Increase the size of an EBS volume on your Mac instance](#mac-instance-increase-volume)
+ [Stop and terminate your Mac instance](#mac-instance-stop)
+ [Subscribe to macOS AMI notifications](#subscribe-notifications)
+ [Release the Dedicated Host for your Mac instance](#mac-instance-release-dedicated-host)

## Considerations<a name="mac-instance-considerations"></a>

The following considerations apply to Mac instances:
+ Mac instances are available only as bare metal instances on [Dedicated Hosts](dedicated-hosts-overview.md), with a minimum allocation period of 24 hours before you can release the Dedicated Host\. You can launch one Mac instance per Dedicated Host\. You can share the Dedicated Host with the AWS accounts or organizational units within your AWS organization, or the entire AWS organization\.
+ Mac instances are available only as On\-Demand Instances\. They are not available as Spot Instances or Reserved Instances\. You can save money on Mac instances by purchasing a [Savings Plan](https://docs.aws.amazon.com/savingsplans/latest/userguide/)\.
+ Mac instances can run one of the following operating systems:
  + macOS Mojave \(version 10\.14\) \(x86 Mac Instances only\)
  + macOS Catalina \(version 10\.15\) \(x86 Mac Instances only\)
  + macOS Big Sur \(version 11\)
  + macOS Monterey \(version 12\)
  + macOS Ventura \(version 13\)
+ EBS hotplug is supported\.
+ AWS does not manage or support the internal SSD on the Apple hardware\. We strongly recommend that you use Amazon EBS volumes instead\. EBS volumes provide the same elasticity, availability, and durability benefits on Mac instances as they do on any other EC2 instance\.
+ We recommend using General Purpose SSD \(`gp2` and `gp3`\) and Provisioned IOPS SSD \(`io1` and `io2`\) with Mac instances for optimal EBS performance\.
+ [Mac instances support Amazon EC2 Auto Scaling\.](http://aws.amazon.com/blogs/compute/implementing-autoscaling-for-ec2-mac-instances/) 
+ On x86 Mac instances, automatic software updates are disabled\. We recommend that you apply updates and test them on your instance before you put the instance into production\. For more information, see [Update the operating system and software](#mac-instance-updates)\.
+ When you stop or terminate a Mac instance, a scrubbing workflow is performed on the Dedicated Host\. For more information, see [Stop and terminate your Mac instance](#mac-instance-stop)\.

**Warning**  
Do not use FileVault\. If data\-at\-rest and data\-in\-transit is required, use [EBS encryption](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html) to avoid boot issues and performance impact\. Enabling FileVault will result in the host failing to boot due to the partitions being locked\.

## Instance readiness<a name="mac-instance-readiness"></a>

After you launch a Mac instance, you'll need to wait until the instance is ready before you can connect to it\. For an AWS vended AMI with a x86 Mac instance or a M1 Mac instance, the launch time can range from approximately 6 minutes to 20 minutes\. Depending on the chosen Amazon EBS volume sizes, the inclusion of additional scripts to *user data*, or additional loaded software on a custom macOS AMI, the launch time may increase\.

You can use a small shell script, like the one below, to poll the describe\-instance\-status API to know when the instance is ready to be connected to\. In the following command, replace the example instance ID with your own\.

```
for i in seq 1 200; do aws ec2 describe-instance-status --instance-ids=i-0123456789example \
    --query='InstanceStatuses[0].InstanceStatus.Status'; sleep 5; done;
```

## Launch a Mac instance<a name="mac-instance-launch"></a>

EC2 Mac instances require a [Dedicated Host](dedicated-hosts-overview.md)\. You first need to allocate a host to your account, and then launch the instance onto the host\.

You can launch a Mac instance using the AWS Management Console or the AWS CLI\. 

### Launch a Mac instance using the console<a name="mac-instance-launch-console"></a>

**To launch a Mac instance onto a Dedicated Host**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. Allocate the Dedicated Host, as follows:

   1. In the navigation pane, choose **Dedicated Hosts**\.

   1. Choose **Allocate Dedicated Host** and then do the following:

      1. For **Instance family**, choose **mac1** or **mac2**\. If the instance family doesn’t appear in the list, it’s not supported in the currently selected Region\.

      1. For **Instance type**, choose **mac1\.metal** or **mac2\.metal** based on the instance family chosen\.

      1. For **Availability Zone**, choose the Availability Zone for the Dedicated Host\.

      1. For **Quantity**, keep **1**\.

      1. Choose **Allocate**\.

1. Launch the instance on the host, as follows:

   1. Select the Dedicated Host that you created and then do the following:

      1. Choose **Actions**, **Launch instance\(s\) onto host**\.

      1. Under **Application and OS Images \(Amazon Machine Image\)**, select a macOS AMI\.

      1. Under **Instance type**, select the appropriate instance type \(**mac1\.metal** or **mac2\.metal**\)\.

      1. Under **Advanced details**, verify that **Tenancy**, **Tenancy host by**, and **Tenancy host ID** are preconfigured based on the Dedicated Host you created\. Update **Tenancy affinity** as needed\.

      1. Complete the wizard, specifying EBS volumes, security groups, and key pairs as needed\.

      1. In the **Summary** panel, choose **Launch instance**\.

   1. A confirmation page lets you know that your instance is launching\. Choose **View all instances** to close the confirmation page and return to the console\. The initial state of an instance is `pending`\. The instance is ready when its state changes to `running` and it passes status checks\.

### Launch a Mac instance using the AWS CLI<a name="mac-instance-launch-cli"></a>

**Allocate the Dedicated Host**

Use the following [allocate\-hosts](https://docs.aws.amazon.com/cli/latest/reference/ec2/allocate-hosts.html) command to allocate a Dedicated Host for your Mac instance, replacing the `instance-type` with either `mac1.metal` or `mac2.metal`, and the `region` and `availability-zone` with the appropriate ones for your environment\.

```
aws ec2 allocate-hosts --region us-east-1 --instance-type mac1.metal --availability-zone us-east-1b --auto-placement "on" --quantity 1
```

**Launch the instance on the host**

Use the following [run\-instances](https://docs.aws.amazon.com/cli/latest/reference/ec2/run-instances.html) command to launch a Mac instance, again replacing the `instance-type` with either `mac1.metal` or `mac2.metal`, and the `region` and `availability-zone` with the ones used previously\.

```
aws ec2 run-instances --region us-east-1 --instance-type mac1.metal --placement Tenancy=host --image-id ami_id --key-name my-key-pair
```

The initial state of an instance is `pending`\. The instance is ready when its state changes to `running` and it passes status checks\. Use the following [describe\-instance\-status](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-instance-status.html) command to display status information for your instance\.

```
aws ec2 describe-instance-status --instance-ids i-017f8354e2dc69c4f
```

The following is example output for an instance that is running and has passed status checks\.

```
{
    "InstanceStatuses": [
        {
            "AvailabilityZone": "us-east-1b",
            "InstanceId": "i-017f8354e2dc69c4f",
            "InstanceState": {
                "Code": 16,
                "Name": "running"
            },
            "InstanceStatus": {
                "Details": [
                    {
                        "Name": "reachability",
                        "Status": "passed"
                    }
                ],
                "Status": "ok"
            },
            "SystemStatus": {
                "Details": [
                    {
                        "Name": "reachability",
                        "Status": "passed"
                    }
                ],
                "Status": "ok"
            }
        }
    ]
}
```

## Connect to your Mac instance<a name="connect-to-mac-instance"></a>

You can connect to your Mac instance using SSH or Apple Remote Desktop \(ARD\)\.

### Connect to your instance using SSH<a name="mac-instance-ssh"></a>

**Important**  
Multiple users can access the OS simultaneously\. Typically there is a 1:1 user:GUI session due to the built\-in Screen Sharing service on port 5900\. Using SSH within macOS supports multiple sessions up until the "Max Sessions" limit in the sshd\_config file\.

Amazon EC2 Mac instances do not allow remote root SSH by default\. Password authentication is disabled to prevent brute\-force password attacks\. The ec2\-user account is configured to log in remotely using SSH\. The ec2\-user account also has sudo privileges\. After you connect to your instance, you can add other users\.

To support connecting to your instance using SSH, launch the instance using a key pair and a security group that allows SSH access, and ensure that the instance has internet connectivity\. You provide the `.pem` file for the key pair when you connect to the instance\.

Use the following procedure to connect to your Mac instance using an SSH client\. If you receive an error while attempting to connect to your instance, see [Troubleshoot connecting to your instance](TroubleshootingInstancesConnecting.md)\.

**To connect to your instance using SSH**

1. Verify that your local computer has an SSH client installed by entering ssh at the command line\. If your computer doesn't recognize the command, search for an SSH client for your operating system and install it\.

1. Get the public DNS name of your instance\. Using the Amazon EC2 console, you can find the public DNS name on both the **Details** and the **Networking** tabs\. Using the AWS CLI, you can find the public DNS name using the [describe\-instances](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-instances.html) command\.

1. Locate the `.pem` file for the key pair that you specified when you launched the instance\.

1. Connect to your instance using the following ssh command, specifying the public DNS name of the instance and the `.pem` file\.

   ```
   ssh -i /path/key-pair-name.pem ec2-user@instance-public-dns-name
   ```

### Connect to your instance using Apple Remote Desktop<a name="mac-instance-vnc"></a>

Use the following procedure to connect to your instance using Apple Remote Desktop \(ARD\)\.

**Note**  
macOS 10\.14 and later only allows control if Screen Sharing is enabled through [System Preferences](https://support.apple.com/guide/remote-desktop/enable-remote-management-apd8b1c65bd/mac)\.

**To connect to your instance using ARD client or VNC client**

1. Verify that your local computer has an ARD client or a VNC client that supports ARD installed\. On macOS, you can leverage the built\-in Screen Sharing application\. Otherwise, search for ARD for your operating system and install it\.

1. From your local computer, [connect to your instance using SSH](#mac-instance-ssh)\.

1. Set up a password for the ec2\-user account using the passwd command as follows\.

   ```
   [ec2-user ~]$ sudo passwd ec2-user
   ```

1. Start the Apple Remote Desktop agent and enable remote desktop access as follows\.

   ```
   [ec2-user ~]$ sudo /System/Library/CoreServices/RemoteManagement/ARDAgent.app/Contents/Resources/kickstart \
   -activate -configure -access -on \
   -restart -agent -privs -all
   ```

1. Disconnect from your instance by typing **exit** and pressing Enter\.

1. From your computer, connect to your instance using the following ssh command\. In addition to the options shown in the previous section, use the \-L option to enable port forwarding and forward all traffic on local port 5900 to the ARD server on the instance\.

   ```
   ssh -L 5900:localhost:5900 -i /path/key-pair-name.pem ec2-user@instance-public-dns-name
   ```

1. From your local computer, use the ARD client or VNC client that supports ARD to connect to `localhost:5900`\. For example, use the Screen Sharing application on macOS as follows:

   1. Open **Finder** and select **Go**\.

   1. Select **Connect to Server**\.

   1. In the **Server Address** field, enter `vnc://localhost:5900`\.

   1. Log in as prompted, using **ec2\-user** as the user name and the password that you created for the ec2\-user account\.

## Modify macOS screen resolution on Mac instances<a name="mac-screen-resolution"></a>

After you connect to your EC2 Mac instance using ARD or a VNC client that supports ARD installed, you can modify the screen resolution of your macOS environment using any of the publicly available macOS tools or utilities, such as [displayplacer](https://github.com/jakehilborn/displayplacer)\.

**Note**  
The current build of displayplacer is not supported on M1 Mac instances\.

**To modify the screen resolution using displayplacer**

1. Install displayplacer\.

   ```
   brew tap jakehilborn/jakehilborn && brew install displayplacer
   ```

1. Show the current screen information and possible screen resolutions\.

   ```
   displayplacer list
   ```

1. Apply the desired screen resolution\.

   ```
   displayplacer "id:<screenID> res:<width>x<height> origin:(0,0) degree:0"
   ```

   For example:

   ```
   RES="2560x1600"
   displayplacer "id:69784AF1-CD7D-B79B-E5D4-60D937407F68 res:${RES} scaling:off origin:(0,0) degree:0"
   ```

## EC2 macOS AMIs<a name="ec2-macos-images"></a>

Amazon EC2 macOS is designed to provide a stable, secure, and high\-performance environment for developer workloads running on Amazon EC2 Mac instances\. EC2 macOS AMIs includes packages that enable easy integration with AWS, such as launch configuration tools and popular AWS libraries and tools\.

EC2 macOS AMIs include the following by default:
+ ENA drivers
+ EC2 macOS Init
+ SSM Agent for macOS
+ EC2 System Monitoring for macOS \(x86 Mac instances only\)
+ AWS Command Line Interface \(AWS CLI\) version 2
+ Command Line Tools for Xcode
+ Homebrew

AWS provides updated EC2 macOS AMIs on a regular basis that include updates to packages owned by AWS and the latest fully\-tested macOS version\. Additionally, AWS provides updated AMIs with the latest minor version updates or major version updates as soon as they can be fully tested and vetted\. If you do not need to preserve data or customizations to your Mac instances, you can get the latest updates by launching a new instance using the current AMI and then terminating the previous instance\. Otherwise, you can choose which updates to apply to your Mac instances\.

## Update the operating system and software<a name="mac-instance-updates"></a>

**Warning**  
Do not install beta or pre\-release macOS versions on your EC2 Mac instances, as this configuration is currently not supported\. Installing beta or pre release macOS versions will lead to degradation of your EC2 Mac Dedicated Host when you stop or terminate your instance, and will prevent you from starting or launching a new instance on that host\. 

**Topics**
+ [Update software on x86 Mac instances](#x86-mac1)
+ [Update software on M1 Mac instances](#mac2)

### Update software on x86 Mac instances<a name="x86-mac1"></a>

On x86 Mac instances, you can install operating system updates from Apple using the `softwareupdate` command\.

****To install operating system updates from Apple on x86 Mac instances****

1. List the packages with available updates using the following command\.

   ```
   [ec2-user ~]$ softwareupdate --list
   ```

1. Install all updates or only specific updates\. To install specific updates, use the following command\.

   ```
   [ec2-user ~]$ sudo softwareupdate --install label
   ```

   To install all updates instead, use the following command\.

   ```
   [ec2-user ~]$ sudo softwareupdate --install --all --restart
   ```

System administrators can use AWS Systems Manager to roll out pre\-approved operating system updates on x86 Mac instances\. For more information, see the [AWS Systems Manager User Guide](https://docs.aws.amazon.com/systems-manager/latest/userguide/)\.

You can use Homebrew to install updates to packages in the EC2 macOS AMIs, so that you have the latest version of these packages on your instances\. You can also use Homebrew to install and run common macOS applications on Amazon EC2 macOS\. For more information, see the [Homebrew Documentation](https://docs.brew.sh/)\.

**To install updates using Homebrew**

1. Update Homebrew using the following command\.

   ```
   [ec2-user ~]$ brew update
   ```

1. List the packages with available updates using the following command\.

   ```
   [ec2-user ~]$ brew outdated
   ```

1. Install all updates or only specific updates\. To install specific updates, use the following command\.

   ```
   [ec2-user ~]$ brew upgrade package name
   ```

   To install all updates instead, use the following command\.

   ```
   [ec2-user ~]$ brew upgrade
   ```

### Update software on M1 Mac instances<a name="mac2"></a>

#### Considerations<a name="mac2-ena-update"></a>

**Elastic Network Adapter \(ENA\) driver**  
Due to an update in the network driver configuration, ENA driver version 1\.0\.2 isn't compatible with macOS 13\.3 or greater\. If you want to update to macOS 13\.3 from an older version and have not installed the latest ENA driver, use the following procedure to install a new version of the driver\.

**To install a new version of the ENA driver**

1. In a Terminal window, connect to your M1 Mac instance using [SSH](#mac-instance-ssh)\.

1. Download the ENA application into the `Applications` file using the following command\.

   ```
   brew install amazon-ena-ethernet-dext
   ```
**Troubleshooting tip**  
If you receive the warning `No available formula with the name amazon-ena-ethernet-dext`, run the following command\.  

   ```
   brew update
   ```

1. Disconnect from your instance by typing **exit** and pressing return\.

1. Use the VNC client to activate the ENA application\.

   1. Setup the VNC client using [Connect to your instance using Apple Remote Desktop](#mac-instance-vnc)\.

   1. Once you have connected to your instance using the Screen Sharing application, go to the **Applications** folder and open the ENA application\. 

   1. Choose **Activate**

   1. To confirm the driver was activated correctly, run the following command in the Terminal window\. The output of the command shows that the old driver is in the terminating state and the new driver is in the activated state\.

      ```
      systemextensionsctl list;
      ```

   1. After you restart the instance, only the new driver will be present\.

#### Software update on M1 Mac instances<a name="mac2-software-update"></a>

On M1 Mac instances, you must complete several steps to perform an in\-place operating system update\. First, access the internal disk of the instance using the GUI with a VNC \(Virtual Network Computing\) client\. This procedure uses macOS Screen Sharing, the built in VNC client\. Then, delegate ownership to the administrative user \(`ec2-user`\) by signing in as `aws-managed-user` on the Amazon EBS volume\.

As you work through this procedure, you create two passwords\. One password is for the administrative user \(`ec2-user`\) and the other password is for a special administrative user \(`aws-managed-user`\)\. Remember these passwords since you will use them as you work through the procedure\.

**Note**  
With this procedure on macOS Big Sur, you can only perform minor updates such as updating from macOS Big Sur 11\.7\.3 to macOS Big Sur 11\.7\.4\. For macOS Monterey or above, you can perform major software updates\.

##### <a name="access-internal-disk"></a>

****To access the internal disk****

1. From your local computer, in the Terminal, connect to your M1 Mac instance using SSH with the following command\. For more information, see [Connect to your instance using SSH](#mac-instance-ssh)\.

   ```
   ssh -i /path/key-pair-name.pem ec2-user@instance-public-dns-name
   ```

1. Install and start MacOS Screen Sharing using the following command\.

   ```
   sudo defaults write /var/db/launchd.db/com.apple.launchd/overrides.plist com.apple.screensharing -dict Disabled -bool false && 
   sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.screensharing.plist
   ```

1. Set a password for `ec2-user` with the following command\. Remember the password as you will use it later\.

   ```
   sudo /usr/bin/dscl . -passwd /Users/ec2-user
   ```

1. Disconnect from the instance by typing **exit** and pressing return\.

1. From your local computer, in the Terminal, reconnect to your instance with an SSH tunnel to the VNC port using the following command\.

   ```
   ssh -i /path/key-pair-name.pem -L 5900:localhost:5900 ec2-user@instance-public-dns-name
   ```
**Note**  
Do not exit this SSH session until the following VNC connection and GUI steps are completed\. When the instance is restarted, the connection will close automatically\.

1. From your local computer, connect to `localhost:5900` using the following steps:

   1. Open **Finder** and select **Go**\.

   1. Select **Connect to Server**\.

   1. In the **Server Address** field, enter `vnc://localhost:5900`\.

1. In the macOS window, connect to the remote session of the M1 Mac instance as `ec2-user` with the password you created in [Step 3](#passwd-step)\.

1. Access the internal disk, named **InternalDisk**, using one of the following options\.

   1. For macOS Ventura or above: Open **System Settings**, select **General** in the left pane, then **Startup Disk** at the lower right of the pane\.

   1. For macOS Monterey or below: Open **System Preferences**, select **Startup Disk**, then unlock the pane by choosing the lock icon in the lower left of the window\.
**Troubleshooting tip**  
If you need to mount the internal disk, run the following command in the Terminal\.  

   ```
   APFSVolumeName="InternalDisk" ; SSDContainer=$(diskutil list | grep "Physical Store disk0" -B 3 | grep "/dev/disk" | awk {'print $1'} ) ; diskutil apfs addVolume $SSDContainer APFS $APFSVolumeName
   ```

1. Choose the internal disk, named **InternalDisk**, and select **Restart**\. Select **Restart** again when prompted\.
**Important**  
If the internal disk is named **Macintosh HD** instead of **InternalDisk**, your instance needs to be stopped and restarted so the dedicated host can be updated\. For more information, see [Stop and terminate your Mac instance](#mac-instance-stop)\.

##### <a name="delegate-access"></a>

Use the following procedure to delegate ownership to the administrative user\. When you reconnect to your instance with SSH, you boot from the internal disk using the special administrative user \(`aws-managed-user`\)\. The initial password for `aws-managed-user` is blank, so you need to overwrite it on your first connection\. Then, you need to repeat the steps to install and start macOS Screen Sharing since the boot volume has changed\.

**To delegate ownership to the administrator on an Amazon EBS volume**

1. From your local computer, in the Terminal, connect to your M1 Mac instance using the following command\. 

   ```
   ssh -i /path/key-pair-name.pem aws-managed-user@instance-public-dns-name
   ```

1. When you receive the warning `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!`, use one of the following commands to resolve this issue\.

   1. Clear out the known hosts using the following command\. Then, repeat the previous step\.

      ```
      rm ~/.ssh/known_hosts
      ```

   1. Add the following to the SSH command in the previous step\.

      ```
      -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no
      ```

1. Set the password for `aws-managed-user` with the following command\. The `aws-managed-user` initial password is blank, so you need to overwrite it on your first connection\.

   1. 

      ```
      sudo /usr/bin/dscl . -passwd /Users/aws-managed-user password
      ```

   1. When you receive the prompt, `Permission denied. Please enter user's old password:`, press enter\.
**Troubleshooting tip**  
If you get the error `passwd: DS error: eDSAuthFailed`, use the following command\.  

      ```
      sudo passwd aws-managed-user
      ```

1. Install and start macOS Screen Sharing using the following command\.

   ```
   sudo defaults write /var/db/launchd.db/com.apple.launchd/overrides.plist com.apple.screensharing -dict Disabled -bool false && 
   sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.screensharing.plist
   ```

1. Disconnect from the instance by typing **exit** and pressing return\.

1. From your local computer, in the Terminal, reconnect to your instance with an SSH tunnel to the VNC port using the following command\.

   ```
   ssh -i /path/key-pair-name.pem -L 5900:localhost:5900 aws-managed-user@instance-public-dns-name
   ```

1. From your local computer, connect to `localhost:5900` using the following steps:

   1. Open **Finder** and select **Go**\.

   1. Select **Connect to Server**\.

   1. In the **Server Address** field, enter `vnc://localhost:5900`\.

1.  In the macOS window, connect to the remote session of the M1 Mac instance as `aws-managed-user` with the password you created in [Step 3](#amu-passwd)\.
**Note**  
When prompted to sign in with your Apple ID, select **Set Up Later**\.

1. Access the Amazon EBS volume using one of the following options\.

   1. For macOS Ventura or above: Open **System Settings**, select **General** in the left pane, then **Startup Disk** at the lower right of the pane\.

   1. For macOS Monterey or below: Open **System Preferences**, select **Startup Disk**, then unlock the pane using the lock icon in the lower left of the window\.
**Note**  
Until the reboot takes place, when prompted for an administrator password, use the password you set above for `aws-managed-user`\. This password may be different from the one you set for `ec2-user` or the default administrator account on your instance\. The following instructions specify when to use your instance's administrator password\.

1. Select the Amazon EBS volume \(the volume not named **InternalDisk** in the **Startup Disk** window\) and choose **Restart**\.
**Note**  
If you have multiple bootable Amazon EBS volumes attached to your M1 Mac instance, be sure to use a unique name for each volume\.

1. Confirm the restart, then choose **Authorize Users** when prompted\.

1. On the **Authorize user on this volume** pane, verify that the administrative user \(`ec2-user` by default\) is selected, then select **Authorize**\.

1. Enter the `ec2-user` password you created in [Step 3](#passwd-step) of the previous procedure, then select **Continue**\.

1. Enter the password for the special administrative user \(`aws-managed-user`\) when prompted\.

1. From your local computer, in the Terminal, reconnect to your instance using SSH with user name `ec2-user`\.
**Troubleshooting tip**  
If you get the warning `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!`, run the following command and reconnect to your instance using SSH\.  

   ```
   rm ~/.ssh/known_hosts
   ```

1. To perform the software update, use the commands under [Update software on x86 Mac instances](#x86-mac1)\.

## EC2 macOS Init<a name="ec2-macos-init"></a>

EC2 macOS Init is used to initialize EC2 Mac instances at launch\. It uses priority groups to run logical groups of tasks at the same time\.

The launchd plist file is `/Library/LaunchDaemons/com.amazon.ec2.macos-init.plist`\. The files for EC2 macOS Init are located in `/usr/local/aws/ec2-macos-init`\.

For more information, see [https://github\.com/aws/ec2\-macos\-init](https://github.com/aws/ec2-macos-init)\.

## EC2 System Monitoring for macOS<a name="mac-instance-system-monitor"></a>

EC2 System Monitoring for macOS provides CPU utilization metrics to Amazon CloudWatch\. It sends these metrics to CloudWatch over a custom serial device in 1\-minute periods\. You can enable or disable this agent as follows\. It is enabled by default\.

```
sudo setup-ec2monitoring [enable | disable]
```

**Note**  
EC2 System Monitoring for macOS is not currently supported on M1 Mac instances\.

## Increase the size of an EBS volume on your Mac instance<a name="mac-instance-increase-volume"></a>

You can increase the size of your Amazon EBS volumes on your Mac instance\. For more information, see [Amazon EBS Elastic Volumes](ebs-modify-volume.md)\.

After you increase the size of the volume, you must increase the size of your APFS container as follows\.

**Make increased disk space available for use**

1. Determine if a restart is required\. If you resized an existing EBS volume on a running Mac instance, you must [reboot](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-reboot.html) the instance to make the new size available\. If disk space modification was done during launch time, a reboot will not be required\.

   View current status of disk sizes: 

   ```
    [ec2-user ~]$ diskutil list external physical
   /dev/disk0 (external, physical):
      #:                       TYPE NAME                    SIZE       IDENTIFIER
      0:                 GUID_partition_scheme            *322.1 GB     disk0
      1:                 EFI EFI                           209.7 MB     disk0s1
      2:                 Apple_APFS Container disk2        321.9 GB     disk0s2
   ```

1. Copy and paste the following command\.

   ```
   PDISK=$(diskutil list physical external | head -n1 | cut -d" " -f1)
   APFSCONT=$(diskutil list physical external | grep "Apple_APFS" | tr -s " " | cut -d" " -f8)
   yes | sudo diskutil repairDisk $PDISK
   ```

1. Copy and paste the following command\.

   ```
   sudo diskutil apfs resizeContainer $APFSCONT 0
   ```

## Stop and terminate your Mac instance<a name="mac-instance-stop"></a>

When you stop a Mac instance, the instance remains in the `stopping` state for about 15 minutes before it enters the `stopped` state\.

When you stop or terminate a Mac instance, Amazon EC2 performs a scrubbing workflow on the underlying Dedicated Host to erase the internal SSD, to clear the persistent NVRAM variables, and to update to the latest device firmware\. This ensures that Mac instances provide the same security and data privacy as other EC2 Nitro instances\. It also enables you to run the latest macOS AMIs\. During the scrubbing workflow, the Dedicated Host temporarily enters the pending state\. On x86 Mac instances, the scrubbing workflow may take up to 50 minutes to complete\. On M1 Mac instances, the scrubbing workflow may take up to 110 minutes to complete\. Additionally, on x86 Mac instances, if the device firmware needs to be updated, the scrubbing workflow may take up to 3 hours to complete\.

You can't start the stopped Mac instance or launch a new Mac instance until after the scrubbing workflow completes, at which point the Dedicated Host enters the `available` state\.

Metering and billing is paused when the Dedicated Host enters the `pending` state\. You are not charged for the duration of the scrubbing workflow\.

## Subscribe to macOS AMI notifications<a name="subscribe-notifications"></a>

To be notified when new AMIs are released or when bridgeOS has been updated, subscribe for notifications using Amazon SNS\.

**To subscribe to macOS AMI notifications**

1. Open the Amazon SNS console at [https://console\.aws\.amazon\.com/sns/v3/home](https://console.aws.amazon.com/sns/v3/home)\.

1. In the navigation bar, change the Region to **US East \(N\. Virginia\)**, if necessary\. You must use this Region because the SNS notifications that you are subscribing to were created in this Region\.

1. In the navigation pane, choose **Subscriptions**\.

1. Choose **Create subscription**\.

1. For the **Create subscription** dialog box, do the following:

   1. For **Topic ARN**, copy and paste one of the following Amazon Resource Names \(ARNs\):
      + **arn:aws:sns:us\-east\-1:898855652048:amazon\-ec2\-macos\-ami\-updates**
      + **arn:aws:sns:us\-east\-1:898855652048:amazon\-ec2\-bridgeos\-updates**

      For **Protocol**:

   1. **Email:**

      For **Endpoint**, type an email address that you can use to receive the notifications\. After you create your subscription you'll receive a confirmation message with the subject line `AWS Notification - Subscription Confirmation`\. Open the email and choose **Confirm subscription** to complete your subscription

   1. **SMS:**

      For **Endpoint**, type a phone number that you can use to receive the notifications\.

   1. **AWS Lambda, Amazon SQS, Amazon Kinesis Data Firehose** \(*Notifications come in JSON format*\):

      For **Endpoint**, enter the ARN for the Lambda function, SQS queue, or Firehose stream you can use to receive the notifications\.

   1. Choose **Create subscription**\.

Whenever macOS AMIs are released, we send notifications to the subscribers of the `amazon-ec2-macos-ami-updates` topic\. Whenever bridgeOS is updated, we send notifications to the subscribers of the `amazon-ec2-bridgeos-updates` topic\. If you no longer want to receive these notifications, use the following procedure to unsubscribe\.

**To unsubscribe from macOS AMI notifications**

1. Open the Amazon SNS console at [https://console\.aws\.amazon\.com/sns/v3/home](https://console.aws.amazon.com/sns/v3/home)\.

1. In the navigation bar, change the Region to **US East \(N\. Virginia\)**, if necessary\. You must use this Region because the SNS notifications were created in this Region\.

1. In the navigation pane, choose **Subscriptions**\.

1. Select the subscriptions and then choose **Actions**, **Delete subscriptions** When prompted for confirmation, choose **Delete**\.

## Release the Dedicated Host for your Mac instance<a name="mac-instance-release-dedicated-host"></a>

When you are finished with your Mac instance, you can release the Dedicated Host\. Before you can release the Dedicated Host, you must stop or terminate the Mac instance\. You cannot release the host until the allocation period exceeds the 24\-hour minimum\.

**To release the Dedicated Host**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Instances**\.

1. Select the instance and choose **Instance state**, then choose either **Stop instance** or **Terminate instance**\.

1. In the navigation pane, choose **Dedicated Hosts**\.

1. Select the Dedicated Host and choose **Actions**, **Release host**\.

1. When prompted for confirmation, choose **Release**\.