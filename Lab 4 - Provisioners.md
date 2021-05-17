# Lab: Packer Provisioners
This lab will walk you through adding a provisoner to your Packer HCL Template. Provisioners use builtin and third-party software to install and configure the machine image after booting.

Duration: 30 minutes

- Task 1: Add a Packer provisioner to install all updates and the nginx service
- Task 2: Validate the Packer Template
- Task 3: Build a new Image using Packer
- Task 4: Install Web App

### Task 1: Update Packer Template to support Multiple Regions
The Packer AWS builder supports the ability to create an AMI in multiple AWS regions.  AMIs are specific to regions so this will ensure that the same image is available in all regions within a single cloud.  We will also leverage Tags to indentify our image.

### Step 1.1.1

Validate that your `aws-ubuntu.pkr.hcl` file has the following Packer `source` block.

```hcl
source "amazon-ebs" "ubuntu" {
  ami_name      = "packer-ubuntu-aws-{{timestamp}}"
  instance_type = "t2.micro"
  region        = "us-west-2"
  ami_regions   = ["us-west-2", "us-east-1", "eu-central-1"]
  source_ami_filter {
    filters = {
      name                = "ubuntu/images/*ubuntu-xenial-16.04-amd64-server-*"
      root-device-type    = "ebs"
      virtualization-type = "hvm"
    }
    most_recent = true
    owners      = ["099720109477"]
  }
  ssh_username = "ubuntu"
  tags = {
    "Name"        = "MyUbuntuImage"
    "Environment" = "Production"
    "OS_Version"  = "Ubuntu 16.04"
    "Release"     = "Latest"
    "Created-by"  = "Packer"
  }
}
```

Add a provisioner to the `builder` block of your `aws-ubuntu.pkr.hcl`

```hcl
build {
  sources = [
    "source.amazon-ebs.ubuntu"
  ]

  provisioner "shell" {
    inline = [
      "echo Installing Updates",
      "sudo apt-get update",
      "sudo apt-get install -y nginx"
    ]
  }
}
```

### Task 2: Validate the Packer Template
Packer templates can be auto formatted and validated via the Packer command line.

### Step 2.1.1

Format and validate your configuration using the `packer fmt` and `packer validate` commands.

```shell
packer fmt aws-ubuntu.pkr.hcl 
packer validate aws-ubuntu.pkr.hcl
```

### Task 3: Build a new Image using Packer
The `packer build` command is used to initiate the image build process for a given Packer template.

### Step 3.1.1
Run a `packer build` for the `aws-ubuntu.pkr.hcl` template.

```shell
packer build aws-ubuntu.pkr.hcl
```

Packer will print output similar to what is shown below.  Note the connection to SSH and the provisioning of the updtes and services before the image is finalized.

```bash
amazon-ebs.ubuntu: output will be in this color.

==> amazon-ebs.ubuntu: Prevalidating any provided VPC information
==> amazon-ebs.ubuntu: Prevalidating AMI Name: packer-ubuntu-aws-1620834314
    amazon-ebs.ubuntu: Found Image ID: ami-0dd273d94ed0540c0
==> amazon-ebs.ubuntu: Creating temporary keypair: packer_609bf80a-7759-bf2c-8ba3-91132538f80f
==> amazon-ebs.ubuntu: Creating temporary security group for this instance: packer_609bf80d-42af-bc1e-0e1f-e914806fb809
==> amazon-ebs.ubuntu: Authorizing access to port 22 from [0.0.0.0/0] in the temporary security groups...
==> amazon-ebs.ubuntu: Launching a source AWS instance...
==> amazon-ebs.ubuntu: Adding tags to source instance
    amazon-ebs.ubuntu: Adding tag: "Name": "Packer Builder"
    amazon-ebs.ubuntu: Instance ID: i-0348c8cc4584f902a
==> amazon-ebs.ubuntu: Waiting for instance (i-0348c8cc4584f902a) to become ready...
==> amazon-ebs.ubuntu: Using ssh communicator to connect: 52.33.179.108
==> amazon-ebs.ubuntu: Waiting for SSH to become available...
==> amazon-ebs.ubuntu: Connected to SSH!
==> amazon-ebs.ubuntu: Provisioning with shell script: /tmp/packer-shell794569831
    amazon-ebs.ubuntu: Installing Updates
    amazon-ebs.ubuntu: Get:1 http://security.ubuntu.com/ubuntu xenial-security InRelease [109 kB]
    amazon-ebs.ubuntu: Hit:2 http://archive.ubuntu.com/ubuntu xenial InRelease
    amazon-ebs.ubuntu: Get:3 http://archive.ubuntu.com/ubuntu xenial-updates InRelease [109 kB]
    amazon-ebs.ubuntu: Get:4 http://security.ubuntu.com/ubuntu xenial-security/main amd64 Packages [1646 kB]
    amazon-ebs.ubuntu: Get:5 http://archive.ubuntu.com/ubuntu xenial-backports InRelease [107 kB]
    amazon-ebs.ubuntu: Get:6 http://archive.ubuntu.com/ubuntu xenial/universe amd64 Packages [7532 kB]
    amazon-ebs.ubuntu: Get:7 http://security.ubuntu.com/ubuntu xenial-security/universe amd64 Packages [786 kB]
    amazon-ebs.ubuntu: Get:8 http://security.ubuntu.com/ubuntu xenial-security/universe Translation-en [226 kB]
    amazon-ebs.ubuntu: Get:9 http://security.ubuntu.com/ubuntu xenial-security/multiverse amd64 Packages [7864 B]
    amazon-ebs.ubuntu: Get:10 http://security.ubuntu.com/ubuntu xenial-security/multiverse Translation-en [2672 B]
    amazon-ebs.ubuntu: Get:11 http://archive.ubuntu.com/ubuntu xenial/universe Translation-en [4354 kB]
    amazon-ebs.ubuntu: Get:12 http://archive.ubuntu.com/ubuntu xenial/multiverse amd64 Packages [144 kB]
    amazon-ebs.ubuntu: Get:13 http://archive.ubuntu.com/ubuntu xenial/multiverse Translation-en [106 kB]
    amazon-ebs.ubuntu: Get:14 http://archive.ubuntu.com/ubuntu xenial-updates/main amd64 Packages [2048 kB]
    amazon-ebs.ubuntu: Get:15 http://archive.ubuntu.com/ubuntu xenial-updates/main Translation-en [482 kB]
    amazon-ebs.ubuntu: Get:16 http://archive.ubuntu.com/ubuntu xenial-updates/universe amd64 Packages [1220 kB]
    amazon-ebs.ubuntu: Get:17 http://archive.ubuntu.com/ubuntu xenial-updates/universe Translation-en [358 kB]
    amazon-ebs.ubuntu: Get:18 http://archive.ubuntu.com/ubuntu xenial-updates/multiverse amd64 Packages [22.6 kB]
    amazon-ebs.ubuntu: Get:19 http://archive.ubuntu.com/ubuntu xenial-updates/multiverse Translation-en [8476 B]
    amazon-ebs.ubuntu: Get:20 http://archive.ubuntu.com/ubuntu xenial-backports/main amd64 Packages [9812 B]
    amazon-ebs.ubuntu: Get:21 http://archive.ubuntu.com/ubuntu xenial-backports/main Translation-en [4456 B]
    amazon-ebs.ubuntu: Get:22 http://archive.ubuntu.com/ubuntu xenial-backports/universe amd64 Packages [11.3 kB]
    amazon-ebs.ubuntu: Get:23 http://archive.ubuntu.com/ubuntu xenial-backports/universe Translation-en [4476 B]
    amazon-ebs.ubuntu: Fetched 19.3 MB in 8s (2362 kB/s)
    amazon-ebs.ubuntu: Reading package lists...
    amazon-ebs.ubuntu: Reading package lists...
    amazon-ebs.ubuntu: Building dependency tree...
    amazon-ebs.ubuntu: Reading state information...
    amazon-ebs.ubuntu: The following additional packages will be installed:
    amazon-ebs.ubuntu:   nginx-common nginx-light
    amazon-ebs.ubuntu: Suggested packages:
    amazon-ebs.ubuntu:   fcgiwrap nginx-doc ssl-cert
    amazon-ebs.ubuntu: The following NEW packages will be installed:
    amazon-ebs.ubuntu:   nginx nginx-common nginx-light
    amazon-ebs.ubuntu: 0 upgraded, 3 newly installed, 0 to remove and 10 not upgraded.
    amazon-ebs.ubuntu: Need to get 345 kB of archives.
    amazon-ebs.ubuntu: After this operation, 1,098 kB of additional disk space will be used.
    amazon-ebs.ubuntu: Get:1 http://security.ubuntu.com/ubuntu xenial-security/main amd64 nginx-common all 1.10.3-0ubuntu0.16.04.5 [26.9 kB]
    amazon-ebs.ubuntu: Get:2 http://security.ubuntu.com/ubuntu xenial-security/universe amd64 nginx-light amd64 1.10.3-0ubuntu0.16.04.5 [315 kB]
    amazon-ebs.ubuntu: Get:3 http://security.ubuntu.com/ubuntu xenial-security/main amd64 nginx all 1.10.3-0ubuntu0.16.04.5 [3,494 B]
==> amazon-ebs.ubuntu: debconf: unable to initialize frontend: Dialog
==> amazon-ebs.ubuntu: debconf: (Dialog frontend will not work on a dumb terminal, an emacs shell buffer, or without a controlling terminal.)
==> amazon-ebs.ubuntu: debconf: falling back to frontend: Readline
==> amazon-ebs.ubuntu: debconf: unable to initialize frontend: Readline
==> amazon-ebs.ubuntu: debconf: (This frontend requires a controlling tty.)
==> amazon-ebs.ubuntu: debconf: falling back to frontend: Teletype
==> amazon-ebs.ubuntu: dpkg-preconfigure: unable to re-open stdin:
    amazon-ebs.ubuntu: Fetched 345 kB in 0s (651 kB/s)
    amazon-ebs.ubuntu: Selecting previously unselected package nginx-common.
    amazon-ebs.ubuntu: (Reading database ... 51474 files and directories currently installed.)
    amazon-ebs.ubuntu: Preparing to unpack .../nginx-common_1.10.3-0ubuntu0.16.04.5_all.deb ...
    amazon-ebs.ubuntu: Unpacking nginx-common (1.10.3-0ubuntu0.16.04.5) ...
    amazon-ebs.ubuntu: Selecting previously unselected package nginx-light.
    amazon-ebs.ubuntu: Preparing to unpack .../nginx-light_1.10.3-0ubuntu0.16.04.5_amd64.deb ...
    amazon-ebs.ubuntu: Unpacking nginx-light (1.10.3-0ubuntu0.16.04.5) ...
    amazon-ebs.ubuntu: Selecting previously unselected package nginx.
    amazon-ebs.ubuntu: Preparing to unpack .../nginx_1.10.3-0ubuntu0.16.04.5_all.deb ...
    amazon-ebs.ubuntu: Unpacking nginx (1.10.3-0ubuntu0.16.04.5) ...
    amazon-ebs.ubuntu: Processing triggers for ureadahead (0.100.0-19.1) ...
    amazon-ebs.ubuntu: Processing triggers for systemd (229-4ubuntu21.31) ...
    amazon-ebs.ubuntu: Processing triggers for ufw (0.35-0ubuntu2) ...
    amazon-ebs.ubuntu: Processing triggers for man-db (2.7.5-1) ...
    amazon-ebs.ubuntu: Setting up nginx-common (1.10.3-0ubuntu0.16.04.5) ...
    amazon-ebs.ubuntu: debconf: unable to initialize frontend: Dialog
    amazon-ebs.ubuntu: debconf: (Dialog frontend will not work on a dumb terminal, an emacs shell buffer, or without a controlling terminal.)
    amazon-ebs.ubuntu: debconf: falling back to frontend: Readline
    amazon-ebs.ubuntu: Setting up nginx-light (1.10.3-0ubuntu0.16.04.5) ...
    amazon-ebs.ubuntu: Setting up nginx (1.10.3-0ubuntu0.16.04.5) ...
    amazon-ebs.ubuntu: Processing triggers for ureadahead (0.100.0-19.1) ...
    amazon-ebs.ubuntu: Processing triggers for systemd (229-4ubuntu21.31) ...
    amazon-ebs.ubuntu: Processing triggers for ufw (0.35-0ubuntu2) ...
==> amazon-ebs.ubuntu: Stopping the source instance...
    amazon-ebs.ubuntu: Stopping instance
==> amazon-ebs.ubuntu: Waiting for the instance to stop...
==> amazon-ebs.ubuntu: Creating AMI packer-ubuntu-aws-1620834314 from instance i-0348c8cc4584f902a
    amazon-ebs.ubuntu: AMI: ami-040bd66b2e79ccb64
==> amazon-ebs.ubuntu: Waiting for AMI to become ready...
==> amazon-ebs.ubuntu: Copying/Encrypting AMI (ami-040bd66b2e79ccb64) to other regions...
    amazon-ebs.ubuntu: Copying to: eu-central-1
    amazon-ebs.ubuntu: Copying to: us-east-1
    amazon-ebs.ubuntu: Waiting for all copies to complete...
==> amazon-ebs.ubuntu: Adding tags to AMI (ami-00604b26ad22ba5ee)...
==> amazon-ebs.ubuntu: Tagging snapshot: snap-0b508fbc52fdd47a7
==> amazon-ebs.ubuntu: Creating AMI tags
    amazon-ebs.ubuntu: Adding tag: "OS_Version": "Ubuntu 16.04"
    amazon-ebs.ubuntu: Adding tag: "Release": "Latest"
    amazon-ebs.ubuntu: Adding tag: "Created-by": "Packer"
    amazon-ebs.ubuntu: Adding tag: "Environment": "Production"
    amazon-ebs.ubuntu: Adding tag: "Name": "MyUbuntuImage"
==> amazon-ebs.ubuntu: Creating snapshot tags
==> amazon-ebs.ubuntu: Adding tags to AMI (ami-08b5a99dee46eede8)...
==> amazon-ebs.ubuntu: Tagging snapshot: snap-0d7a5c9e418f54157
==> amazon-ebs.ubuntu: Creating AMI tags
    amazon-ebs.ubuntu: Adding tag: "Created-by": "Packer"
    amazon-ebs.ubuntu: Adding tag: "Environment": "Production"
    amazon-ebs.ubuntu: Adding tag: "Name": "MyUbuntuImage"
    amazon-ebs.ubuntu: Adding tag: "OS_Version": "Ubuntu 16.04"
    amazon-ebs.ubuntu: Adding tag: "Release": "Latest"
==> amazon-ebs.ubuntu: Creating snapshot tags
==> amazon-ebs.ubuntu: Adding tags to AMI (ami-040bd66b2e79ccb64)...
==> amazon-ebs.ubuntu: Tagging snapshot: snap-06fd314adc723181b
==> amazon-ebs.ubuntu: Creating AMI tags
    amazon-ebs.ubuntu: Adding tag: "Name": "MyUbuntuImage"
    amazon-ebs.ubuntu: Adding tag: "OS_Version": "Ubuntu 16.04"
    amazon-ebs.ubuntu: Adding tag: "Release": "Latest"
    amazon-ebs.ubuntu: Adding tag: "Created-by": "Packer"
    amazon-ebs.ubuntu: Adding tag: "Environment": "Production"
==> amazon-ebs.ubuntu: Creating snapshot tags
==> amazon-ebs.ubuntu: Terminating the source AWS instance...
==> amazon-ebs.ubuntu: Cleaning up any extra volumes...
==> amazon-ebs.ubuntu: No volumes to clean up, skipping
==> amazon-ebs.ubuntu: Deleting temporary security group...
==> amazon-ebs.ubuntu: Deleting temporary keypair...
Build 'amazon-ebs.ubuntu' finished after 8 minutes 45 seconds.

==> Wait completed after 8 minutes 45 seconds

==> Builds finished. The artifacts of successful builds are:
--> amazon-ebs.ubuntu: AMIs were created:
eu-central-1: ami-08b5a99dee46eede8
us-east-1: ami-00604b26ad22ba5ee
us-west-2: ami-040bd66b2e79ccb64
```


### Task 4: Install Web App

#### Step 4.1.1
Copy the web application assets into our packer working directory

```bash
mkdir assets
cp -R /workstation/terraform/assets/ .
```

#### Step 4.1.2
Replace the existing `builder` block of your `aws-ubuntu.pkr.hcl` with the code below. This will add a `file` provisioner and an additional `shell` provisioner.

```hcl
build {
  sources = [
    "source.amazon-ebs.ubuntu"
  ]

  provisioner "shell" {
    inline = [
      "echo Installing Updates",
      "sudo apt-get update",
    ]
  }

  provisioner "file" {
    source      = "assets"
    destination = "/tmp/"
  }

   provisioner "shell" {
    inline = [
      "sudo sh /tmp/assets/setup-web.sh",
    ]
  }
}
```

Format and validate your configuration using the `packer fmt` and `packer validate` commands.

```shell
packer fmt aws-ubuntu.pkr.hcl 
packer validate aws-ubuntu.pkr.hcl
```

Run a `packer build` for the `aws-ubuntu.pkr.hcl` template.

```shell
packer build aws-ubuntu.pkr.hcl
```
