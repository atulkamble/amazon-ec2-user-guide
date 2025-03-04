# Getting Started with EFAs<a name="efa-start"></a>

In this tutorial, you create an EFA\-enabled AMI and an EFA\-enabled security group, and then launch EFA\-enabled instances into a cluster placement group using that AMI and security group\.

**Topics**
+ [Step 1: Prepare an EFA\-Enabled Security Group](#efa-start-security)
+ [Step 2: Launch a Temporary Instance](#efa-start-tempinstance)
+ [Step 3: Install Libfabric and Open MPI](#efa-start-enable)
+ [Step 4: \(Optional\) Install Intel MPI](#efa-start-impi)
+ [Step 5: Install Your HPC Application](#efa-start-hpc-app)
+ [Step 6: Create an EFA\-Enabled AMI](#efa-start-ami)
+ [Step 7: Launch EFA\-Enabled Instances into a Cluster Placement Group](#efa-start-instances)
+ [Step 8: Terminate the Temporary Instance](#efa-start-terminate)

## Step 1: Prepare an EFA\-Enabled Security Group<a name="efa-start-security"></a>

An EFA requires a security group that allows all inbound and outbound traffic to and from the security group itself\.

**To create an EFA\-enabled security group**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Security Groups** and then choose **Create Security Group**\.

1. In the **Create Security Group** window, do the following:

   1. For **Security group name**, enter a descriptive name for the security group, such as `EFA-enabled security group`\.

   1. \(Optional\) For **Description**, enter a brief description of the security group\.

   1. For **VPC**, select the VPC into which you intend to launch your EFA\-enabled instances\.

   1. Choose **Create**\.

1. Select the security group that you created, and on the **Description** tab, copy the **Group ID**\.

1. On the **Inbound** and **Outbound** tabs, do the following:

   1. Choose **Edit**\.

   1. For **Type**, choose **All traffic**\.

   1. For **Source**, choose **Custom**\.

   1. Paste the security group ID that you copied into the field\.

   1. Choose **Save**\.

## Step 2: Launch a Temporary Instance<a name="efa-start-tempinstance"></a>

Launch a temporary instance that you can use to install and configure the EFA software components\. You use this instance to create an EFA\-enabled AMI from which you can launch your EFA\-enabled instances\.

**To launch a temporary instance**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. Choose **Launch Instance**\.

1. On the **Choose an AMI** page, choose **Select** for one of the following supported AMIs: Amazon Linux, Amazon Linux 2, Red Hat Enterprise Linux 7\.6, CentOS 7\.6, Ubuntu 16\.04, and Ubuntu 18\.04\.

1. On the **Choose an Instance Type** page, select one of the following supported instance types and then choose **Next: Configure Instance Details**: `c5n.18xlarge`, `c5n.metal`, `i3en.24xlarge`, and `p3dn.24xlarge`\.

1. On the **Configure Instance Details** page, do the following:

   1. For **Elastic Fabric Adapter**, choose **Enable**\.

   1. In the **Network Interfaces** section, for device **eth0**, choose **New network interface**\.

   1. Choose **Next: Add Storage**\.

1. On the **Add Storage** page, specify the volumes to attach to the instances in addition to the volumes specified by the AMI \(such as the root device volume\), and then choose **Next: Add Tags**\.

1. On the **Add Tags** page, specify a tag that you can use to identify the temporary instance, and then choose **Next: Configure Security Group**\.

1. On the **Configure Security Group** page, for **Assign a security group**, select **Select an existing security group**, and then select the security group that you created in **Step 1\.**

1. On the **Review Instance Launch** page, review the settings, and then choose **Launch** to choose a key pair and to launch your instance\.

## Step 3: Install Libfabric and Open MPI<a name="efa-start-enable"></a>

Install the EFA\-enabled kernel, EFA drivers, libfabric, and Open MPI stack that is required to support EFA on your temporary instance\.

**To install libfabric and Open MPI on your temporary instance**

1. Connect to the instance you launched in **Step 2**\. For more information, see [Connect to Your Linux Instance](AccessingInstances.md)\.

1. Download the EFA software installation files\. To download the latest *stable* version, use the following command\.

   ```
   $ curl -O https://s3-us-west-2.amazonaws.com/aws-efa-installer/aws-efa-installer-1.5.3.tar.gz
   ```
**Note**  
You can also get the latest version by replacing the version number with `latest` in the command above\.

1. The software installation files are packaged into a compressed `.tar.gz` file\. Extract the files from the compressed `.tar.gz` file and navigate into the extracted directory\.

   ```
   $ tar -xf aws-efa-installer-1.5.3.tar.gz
   ```

   ```
   $ cd aws-efa-installer
   ```

1. Run the EFA software installation script\.

   ```
   $ sudo ./efa_installer.sh -y
   ```

   Libfabric is installed in the `/opt/amazon/efa` directory, while Open MPI is installed in the `/opt/amazon/openmpi` directory\.

1. Log out of the instance and then log back in\.

1. Confirm that the EFA software components were successfully installed\.

   ```
   $ fi_info -p efa
   ```

   The command should return information about the libfabric EFA interfaces\. The following example shows the command output\.

   ```
   provider: efa
       fabric: EFA-fe80::94:3dff:fe89:1b70
       domain: efa_0-rdm
       version: 2.0
       type: FI_EP_RDM
       protocol: FI_PROTO_EFA
   provider: efa
       fabric: EFA-fe80::94:3dff:fe89:1b70
       domain: efa_0-dgrm
       version: 2.0
       type: FI_EP_DGRAM
       protocol: FI_PROTO_EFA
   provider: efa;ofi_rxd
       fabric: EFA-fe80::94:3dff:fe89:1b70
       domain: efa_0-dgrm
       version: 1.0
       type: FI_EP_RDM
       protocol: FI_PROTO_RXD
   ```

## Step 4: \(Optional\) Install Intel MPI<a name="efa-start-impi"></a>

**Note**  
If you intend to use Open MPI, skip this step\. Perform this step only if you intend to use Intel MPI\.

Intel MPI installation requires an additional installation script and environment variable configuration\.

### Prerequisites<a name="efa-start-impi-prereq"></a>

Ensure that the user performing the following steps has sudo privileges\.

**To install Intel MPI**

1. Download the Intel MPI installation script\.

   ```
   $ curl -O http://registrationcenter-download.intel.com/akdlm/irc_nas/tec/15553/aws_impi.sh
   ```

1. Change the script permissions to add group read/write permissions\.

   ```
   $ chmod 755 aws_impi.sh
   ```

1. Run the installation script\.

   ```
   $ ./aws_impi.sh install
   ```

   Intel MPI is installed in the `/opt/intel/impi/2019.4.243/intel64` directory\.

1. Add the Intel MPI environment variables to the corresponding shell startup scripts to ensure that they are set each time that the instance starts\. Do one of the following depending on your shell\.
**Note**  
The **csh** shell is not supported due to environment variable length limitations\.
   + For **bash**, add the following environment variable to `/home/username/.bashrc` and `/home/username/.bash_profile`\.

     ```
     source /opt/intel/impi/2019.4.243/intel64/bin/mpivars.sh
     ```
   + For **tcsh**, add the following environment variable to `/home/username/.cshrc`\.

     ```
     source /opt/intel/impi/2019.4.243/intel64/bin/mpivarsh.csh
     ```

1. Log out of the instance and then log back in\.

1. Run the following command to confirm that Intel MPI was successfully installed\.

   ```
   $ which mpicc
   ```

   The following example shows the command output\.

   ```
   /opt/intel/compilers_and_libraries_2019.4.243/linux/mpi/intel64/bin/mpicc
   ```

**Note**  
If you no longer want to use Intel MPI, remove the environment variables from the shell startup scripts\.

## Step 5: Install Your HPC Application<a name="efa-start-hpc-app"></a>

Install the HPC application on the temporary instance\. The installation procedure varies depending on the specific HPC application\. For more information about installing software on your Linux instance, see [Managing Software on Your Linux Instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/managing-software.html)\.

**Note**  
You might need to refer to your HPC application’s documentation for installation instructions\.

## Step 6: Create an EFA\-Enabled AMI<a name="efa-start-ami"></a>

After you have installed the required software components, you create an EFA\-AMI that you can reuse to launch your EFA\-enabled instances\.

**To create an AMI from your temporary instance**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Instances**\.

1. Select the instance that you created in **Step 2** and choose **Actions**, **Image**, **Create Image**\.

1. In the **Create Image** window, do the following:

   1. For **Image name**, enter a descriptive name for the AMI, such as `EFA-enabled AMI`\.

   1. \(Optional\) For **Image description**, enter a brief description of the AMI\.

   1. Choose **Create Image** and then choose **Close**\.

1. In the navigation pane, choose **AMIs**\.

1. Locate the AMI you created in the list\. Wait for the Status to transition from `pending` to `available` before continuing to the next step\.

## Step 7: Launch EFA\-Enabled Instances into a Cluster Placement Group<a name="efa-start-instances"></a>

Launch your EFA\-enabled instances into a cluster placement group using the EFA\-enabled AMI that you created in **Step 5**, and the EFA\-enabled security group that you created in **Step 1**\.

**Note**  
It is not an absolute requirement to launch your EFA\-enabled instances into a cluster placement group\. However, we do recommend running your EFA\-enabled instances in a cluster placement group as it launches the instances into a low\-latency group in a single Availability Zone\.

**To launch your EFA\-enabled instances into a cluster placement group**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. Choose **Launch Instance**\.

1. On the **Choose an AMI** page, choose **My AMIs**, find the AMI that you created in **Step 5**, and then choose **Select**\.

1. On the **Choose an Instance Type** page, select one of the following supported instance types and then choose **Next: Configure Instance Details**: `c5n.18xlarge`, `c5n.metal`, `i3en.24xlarge`, and `p3dn.24xlarge`\.

1. On the **Configure Instance Details** page, do the following:

   1. For **Number of instances**, enter the number of EFA\-enabled instances that you want to launch\.

   1. For **Network** and **Subnet**, select the VPC and subnet into which to launch the instances\.

   1. For **Placement group**, select **Add instance to placement group**\.

   1. For **Placement group name**, select **Add to a new placement group**, enter a descriptive name for the placement group, and then for **Placement group strategy**, select **cluster**\.

   1. For **EFA**, choose **Enable**\.

   1. In the **Network Interfaces** section, for device **eth0**, choose **New network interface**\. You can optionally specify a primary IPv4 address and one or more secondary IPv4 addresses\. If you're launching the instance into a subnet that has an associated IPv6 CIDR block, you can optionally specify a primary IPv6 address and one or more secondary IPv6 addresses\.

   1. Choose **Next: Add Storage**\.

1. On the **Add Storage** page, specify the volumes to attach to the instances in addition to the volumes specified by the AMI \(such as the root device volume\), and then choose **Next: Add Tags**\.

1. On the **Add Tags** page, specify tags for the instances, such as a user\-friendly name, and then choose **Next: Configure Security Group**\.

1. On the **Configure Security Group** page, for **Assign a security group**, select **Select an existing security group**, and then select the security group that you created in **Step 1\.**

1. Choose **Review and Launch**\.

1. On the **Review Instance Launch** page, review the settings, and then choose **Launch** to choose a key pair and to launch your instances\.

## Step 8: Terminate the Temporary Instance<a name="efa-start-terminate"></a>

At this point, you no longer need the temporary instance that you launched in **Step 2**\. You can terminate the instance to stop incurring charges for it\.

**To terminate the temporary instance**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Instances**\.

1. Select the temporary instance that you created in **Step 2** and then choose **Actions**, **Instance State**, **Terminate**, **Yes, Terminate**\.