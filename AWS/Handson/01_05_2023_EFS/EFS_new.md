# Hands-on EFS-01 : How to Create EFS & Attach the EFS to the multiple EC2 Linux 2 Instances

## Outline

- Part 1 - Prep(EC2 SecGrp, EFS SecGrp, EC2 Linux 2 Instance)

- Part 2 - Creating EFS

- Part 3 - Attach the EFS to the multiple EC2 Linux 2 instances

- Part 4 - Auto-mount EFS on Reboot 


## Part 1 - Prep (EC2 SecGrp, EFS SecGrp, EC2 Linux 2 Instance)

### Step 1 - Create EC2 SecGrp:

- Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/.

- Choose the Security Groups on left-hand menu

- Click the `Create Security Group`.

```text
Security Group Name  : EC2 SecGrp
Description          : EC2 SecGrp
VPC                  : Default VPC
Inbound Rules:
    - Type: SSH ----> Source: Anywhere
Outbound Rules: Keep it as default
Tag:
    - Key   : Name
      Value : EC2 SecGrp
```

### Step 2 - Create EFS SecGrp:

- Click the `Create Security Group`.

```text
Security Group Name  : EFS SecGrp
Description          : EFS SecGrp
VPC                  : Default VPC
***Inbound Rules:
    - Type: NFS ----> Port: 2049 ------>Source: sg-EC2 SecGrp
Outbound Rules: Keep it as default
Tag:
    - Key   : Name
      Value : EFS SecGrp
```

### Step 3 - Create EC2 :

- Configure First Instance in N.Virginia

```text
AMI             : Amazon Linux 2
Instance Type   : t2.micro
Network         : default
Subnet          : default
Security Group  : EC2 SecGrp
    Sec.Group Name : EC2 SecGrp
Tag             :
    Key         : Name
    Value       : EC2-1
```

- Configure Second Instance in N.Virginia

```text
AMI             : Amazon Linux 2
Instance Type   : t2.micro
Network         : default
Subnet          : default
Security Group  : EC2 SecGrp
    Sec.Group Name : EC2 SecGrp
Tag             :
    Key         : Name
    Value       : EC2-2
```

## Part 2 - Creating EFS

Open the Amazon EFS console at https://console.aws.amazon.com/efs/.

- Click "Create File System" 

```text

Name                            : FirstEFS
Virtual Private Cloud (VPC)     : Default VPC (Keep default)
Availability and Durability     : Regional (Keep default)
```

- To customize settings manually, select the 'Customize' option seen at the bottom 

```text

General

Name                    : FirstEFS (Comes default from previous setting)

Availability and Durability : Regional (Comes default from previous setting)

Automatic backups       : Unchecked "Enable automatic backups"

Lifecycle management    : Select "None"

Performance mode        : General Purpose

Throughput mode         : Bursting

Encryption              : Enable encryption of data at rest

Tags                    : optional
```
Click Next

```text

- Virtual Private Cloud (VPC): Default VPC

- Mount targets: 
  - select all AZ (keep it default)
  - Clear "default sg" from all AZ
  - Add "EFS SecGrp" to all AZ
  

- Show that you can only add one mount point for each AZ though it has multiple subnets(for example custom VPC) 
```
Click next 

```text
File system policy - optional------> keep it as is
```
Click next. Then review and Create 

Show that it is created. 

## Part 3 - Attach the EFS to the multiple EC2 Linux 2 instances

### STEP-1: Configure the EC2-1 instance


- open EC2 console

-  Connect to EC2-1 with SSH.
```text
ssh -i .....pem ec2-user@..................
```
- Update the installed packages and package cache on your instance.

```text
sudo yum update -y
```
- Change the hostname 

```text
sudo hostnamectl set-hostname First
```

- type "bash" to see new hostname.

```text
bash
```

- Install the "Amazon-efs-utils Package" on Amazon Linux

```text
sudo yum install -y amazon-efs-utils
```

- Create Mounting point 

```text
sudo mkdir efs
```

- Go to the EFS console and click  on "FirstEFS" . Then click "Attach" button seen top of the "EFS" page.

- On the pop up window, copy the script seen under "Using the EFS mount helper" option: "sudo mount -t efs -o tls fs-60d485e2:/ efs"

- Turn back to the terminal and mount EFS using the "EFS mount helper" to the "efs" mounting point

```text
sudo mount -t efs -o tls fs-xxxxxx:/ efs
```
- Check the "efs" folder
```text
ls
```
- Go the "efs" folder and create a new file with Nano editor.

```text
cd efs
sudo nano example.txt
```
- Write something, save and exit;
```text
"hello from first EC2-1"
CTRL X+Y
```


- check the example.txt

```text
cat example.txt
```

- Check the IP of EFS mount point 

```text
netstat -tulpan | grep 2049
```

### STEP-2: Configure the EC2-2 instance


-  Connect to EC2-2 with SSH.
```text
ssh -i .....pem ec2-user@..................
```
- Update the installed packages and package cache on your instance.

```text
sudo yum update -y
```
- Change the hostname 

```text
sudo hostnamectl set-hostname Second
```
- type "bash" to see new hostname.

```text
bash
```
- Install the "Amazon-efs-utils Package" on Amazon Linux

```text
sudo yum install -y amazon-efs-utils
```

- Create Mounting point 

```text
sudo mkdir efs
```

- Go to the EFS console and click  on "FirstEFS" . Then click "Attach" button seen top of the "EFS" page.

- On the pop up window, copy the script seen under "Using the EFS mount helper" option: "sudo mount -t efs -o tls fs-60d485e2:/ efs"

- Turn back to the terminal and mount EFS using the "EFS mount helper" to the "efs" mounting point

```text
sudo mount -t efs -o tls fs-xxxxxxx:/ efs
```
- Check the "efs" folder
```text
ls
```
- Check the example.txt. Show that you can also reach the same file.

```text
cat example.txt
```

- Add something example.txt

```text
sudo nano example.txt
"hello from first EC2-2"
CTRL X+Y
```
- Check the example.txt

```text
cat example.txt

"hello from first EC2-1"
"hello from first EC2-2"
```
- Connect from EC2-1 to the "efs" and show the example.txt:


```text
cd efs
cat example.txt

"hello from first EC2-1"
"hello from first EC2-2"
```
### STEP-3: Configure the EC2-3 instance with EFS while Launching

- go to the EC2 console and click "Launch Instance"

- Configure third Instance in N.Virginia

```text
AMI             : Amazon Linux 2
Instance Type   : t2.micro
Network         : ***Edit >>>>> Choose one of the default subnet
Configure Storage : ***Advanced

                 File systems  >>>> Edit >>>Add file system-------> FirstEFS 
                *** (Note down the mnt point "/mnt/efs/fs1")
Security Group  : EC2 SecGrp
    Sec.Group Name : EC2 SecGrp
Tag             :
    Key         : Name
    Value       : EC2-3
```
- Connect to EC2-3 with SSH

- Change the hostname 

```text
sudo hostnamectl set-hostname Third
```

```text
ssh -i .....pem ec2-user@..................
```
- Go to the directory of mount target 
```text
cd /mnt/efs/fs1/
```
- Show the example.txt:

```text
cd efs
cat example.txt

"hello from first EC2-1"
"hello from first EC2-2"
```
 - Add something example.txt

```text
sudo nano example.txt
"hello from first EC2-3"
CTRL X+Y
```
- Check the example.txt

```text
cat example.txt

"hello from first EC2-1"
"hello from first EC2-2"
"hello from first EC2-3"
```

# PART 4 - AUTOMOUNT EFS ON REBOOT

- Switch to EC2-1 or EC2-2

- reboot and show that configuration is gone for EC2-1 / EC2-2. Since EC2-3 is configured by AWS fstab is set. 
```
sudo reboot now
```
- back up the /etc/fstab file.
```
sudo cp /etc/fstab /etc/fstab.bak
```
- open /etc/fstab file and 

```
sudo nano /etc/fstab 
```
- add the following info to the existing one.
```
 fs-05e00f25c0ae1d5c6:/  /home/ec2-user/efs  efs  defaults,_netdev   0    0

```
- CTRL+X and Y to save

- reboot and show that configuration exists (NOTE)
```
sudo reboot now
```
- list volumes to show current status, all volumes and partitions should be listed
```
lsblk
```
- show the used and available capacities related with volumes and partitions
```
df -h
```
- if there is data on it, check if the data still persists.

- Terminate instances and delete file system from console.
