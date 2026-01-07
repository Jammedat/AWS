The Nautilus DevOps team is tasked with enabling internet access for an EC2 instance running in a private subnet. This instance should be able to upload a test file to a public S3 bucket once it can access the internet. To minimize costs, the team has decided to use a NAT Instance instead of a NAT Gateway.

The following components already exist in the environment:
1) A VPC named xfusion-priv-vpc and a private subnet named xfusion-priv-subnet have been created.
2) An EC2 instance named xfusion-priv-ec2 is already running in the private subnet.
3) The EC2 instance is configured with a cron job that uploads a test file to the S3 bucket xfusion-nat-26783 every minute. Upload will only succeed once internet access is established.

Your task is to:

Create a new public subnet named xfusion-pub-subnet in the existing VPC.
Launch a NAT Instance in the public subnet using an Amazon Linux 2 AMI and name it xfusion-nat-instance. Configure this instance to act as a NAT instance. Make sure to use a custom security group for this instance.
After the configuration, verify that the test file xfusion-test.txt appears in the S3 bucket xfusion-nat-26783. This indicates successful internet access from the private EC2 instance via the NAT Instance.

### Solution
## 1. Create a VPC for the NAT instance

Use the following procedure to create a VPC with a public subnet and a private subnet.

###### To create the VPC

1. Open the Amazon VPC console at <https://console.aws.amazon.com/vpc/>.

2. Choose **Create VPC**.

3. For **Resources to create**, choose **VPC and more**.

4. For **Name tag auto-generation**, enter a name for the VPC.

5. To configure the subnets, do the following:

   1. For **Number of Availability Zones**, choose **1** or **2**, depending on your needs.

   2. For **Number of public subnets**, ensure that you have one public subnet per Availability Zone.

   3. For **Number of private subnets**, ensure that you have one private subnet per Availability Zone.

6. Choose **Create VPC**.

## 2. Create a security group for the NAT instance

Create a security group with the rules described in the following table. These rules enable your NAT instance to receive internet-bound traffic from instances in the private subnet, as well as SSH traffic from your network. The NAT instance can also send traffic to the internet, which enables the instances in the private subnet to get software updates.

The following are the inbound recommended rules.

| Source                                    | Protocol | Port range | Comments                                                                                   |
| ----------------------------------------- | -------- | ---------- | ------------------------------------------------------------------------------------------ |
| `Private subnet CIDR`                     | TCP      | 80         | Allow inbound HTTP traffic from servers in the private subnet                              |
| `Private subnet CIDR`                     | TCP      | 443        | Allow inbound HTTPS traffic from servers in the private subnet                             |
| `Public IP address range of your network` | TCP      | 22         | Allow inbound SSH access to the NAT instance from your network (over the internet gateway) |

The following are the recommended outbound rules.

| Destination | Protocol | Port range | Comments                                    |
| ----------- | -------- | ---------- | ------------------------------------------- |
| 0.0.0.0/0   | TCP      | 80         | Allow outbound HTTP access to the internet  |
| 0.0.0.0/0   | TCP      | 443        | Allow outbound HTTPS access to the internet |

###### To create the security group

1. Open the Amazon VPC console at <https://console.aws.amazon.com/vpc/>.

2. In the navigation pane, choose **Security groups**.

3. Choose **Create security group**.

4. Enter a name and description for the security group.

5. For **VPC**, select the ID of the VPC for your NAT instance.

6. Add rules for inbound traffic under **Inbound rules** as follows:

   1. Choose **Add rule**. Choose **HTTP** for **Type** and enter the IP address range of your private subnet for **Source**.

   2. Choose **Add rule**. Choose **HTTPS** for **Type** and enter the IP address range of your private subnet for **Source**.

   3. Choose **Add rule**. Choose **SSH** for **Type** and enter the IP address range of your network for **Source**.

7. Add rules for outbound traffic under **Outbound rules** as follows:

   1. Choose **Add rule**. Choose **HTTP** for **Type** and enter 0.0.0.0/0 for **Destination**.

   2. Choose **Add rule**. Choose **HTTPS** for **Type** and enter 0.0.0.0/0 for **Destination**.

8. Choose **Create security group**.

For more information, see [Security groups](./vpc-security-groups.html).

## 3. Create a NAT AMI

A NAT AMI is configured to run NAT on an EC2 instance. You must create a NAT AMI and then launch your NAT instance using your NAT AMI.

If you plan to use an operating system other than Amazon Linux for your NAT AMI, refer to the documentation for this operating system to learn how to configure NAT. Be sure to save these settings so that they persist even after an instance reboot.

###### To create a NAT AMI for Amazon Linux

1. Launch an EC2 instance running AL2023 or Amazon Linux 2. Be sure to specify the security group that you created for the NAT instance.

2. Connect to your instance and run the following commands on the instance to enable iptables.

   ```
   sudo yum install iptables-services -y
   sudo systemctl enable iptables
   sudo systemctl start iptables
   ```

3. Do the following on the instance to enable IP forwarding such that it persists after reboot:

   1. Using a text editor, such as **nano** or **vim**, create the following configuration file: `/etc/sysctl.d/custom-ip-forwarding.conf`.

   2. Add the following line to the configuration file.

      ```
      net.ipv4.ip_forward=1
      ```

   3. Save the configuration file and exit the text editor.

   4. Run the following command to apply the configuration file.

      ```
      sudo sysctl -p /etc/sysctl.d/custom-ip-forwarding.conf
      ```

4. Run the following command on the instance, and note the name of the primary network interface. You'll need this information for the next step.

   ```
   netstat -i
   ```

   In the following example output, `docker0` is a network interface created by docker, `eth0` is the primary network interface, and `lo` is the loopback interface.

   ```
   Iface      MTU    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
   docker0   1500        0      0      0 0             0      0      0      0 BMU
   eth0      9001  7276052      0      0 0       5364991      0      0      0 BMRU
   lo       65536   538857      0      0 0        538857      0      0      0 LRU
   ```

   In the following example output, the primary network interface is `enX0`.

   ```
   Iface      MTU    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
   enX0      9001     1076      0      0 0          1247      0      0      0 BMRU
   lo       65536       24      0      0 0            24      0      0      0 LRU
   ```

   In the following example output, the primary network interface is `ens5`.

   ```
   Iface      MTU    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
   ens5      9001    14036      0      0 0          2116      0      0      0 BMRU
   lo       65536       12      0      0 0            12      0      0      0 LRU
   ```

5. Run the following commands on the instance to configure NAT. If the primary network interface is not `eth0`, replace `eth0` with the primary network interface that you noted in the previous step.

   ```
   sudo /sbin/iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
   sudo /sbin/iptables -F FORWARD
   sudo service iptables save
   ```


## 5. Disable source/destination checks

Each EC2 instance performs source/destination checks by default. This means that the instance must be the source or destination of any traffic it sends or receives. However, a NAT instance must be able to send and receive traffic when the source or destination is not itself. Therefore, you must disable source/destination checks on the NAT instance.

###### To disable source/destination checking

1. Open the Amazon EC2 console at <https://console.aws.amazon.com/ec2/>.

2. In the navigation pane, choose **Instances**.

3. Select the NAT instance.

4. Choose **Actions**, **Networking**, **Change source/destination check**.

5. For **Source/destination checking**, select **Stop**.

6. Choose **Save**.

7. If the NAT instance has a secondary network interface, choose it from **Network interfaces** on the **Networking** tab. Choose the interface ID to go to the network interfaces page. Choose **Actions**, **Change source/dest. check**, clear **Enable**, and choose **Save**.

## 6. Update the route table

The route table for the private subnet must have a route that sends internet traffic to the NAT instance.

###### To update the route table

1. Open the Amazon VPC console at <https://console.aws.amazon.com/vpc/>.

2. In the navigation pane, choose **Route tables**.

3. Select the route table for the private subnet.

4. On the **Routes** tab, choose **Edit routes** and then choose **Add route**.

5. Enter 0.0.0.0/0 for **Destination** and the instance ID of the NAT instance for **Target**.

6. Choose **Save changes**.

For more information, see [Configure route tables](./VPC_Route_Tables.html).

## 7. Test your NAT instance

After you have launched a NAT instance and completed the configuration steps above, you can test whether an instance in your private subnet can access the internet through the NAT instance by using the NAT instance as a bastion server.
Check if the file is uploaded to the s3 bucket or not.
