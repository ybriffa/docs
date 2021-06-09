---
title: How to use Terraform
slug: how-to-use-terraform
description: Procedure of use of Terraform
keywords: infrastructure, instance, cloud, creation
excerpt: Step-by-step documentation on how to use Terraform configurations for your infrastructure
section: Tutorials
---

**Last updated 8th June 2021**

## Objective

OpenStack is an open source cloud operating system for building and managing public and private cloud computing platforms. The OpenStack software components are the foundation of the OVHcloud Public Cloud infrastructure.

The open source tool Terraform was developed to make the creation of complex cloud infrastructures easier. It enables your infrastructure's properties to be abstracted in text files which can be used as a basis to deploy your infrastructure.

**This guide explains how to use Terraform with the Public Cloud by way of practical examples.**

### Requirements

- [Configuring user access to Horizon](../configure_user_access_to_horizon/)
- [Preparing an environment for using the OpenStack API](../prepare_the_environment_for_using_the_openstack_api/)
- [Setting OpenStack environment variables](../set-openstack-environment-variables/)
- [Your OVHcloud API identifiers and authorisation key](../../api/first-steps-with-ovh-api/)
- [An SSH key](../create-ssh-keys)
- [Terraform package or binaries](https://www.terraform.io/intro/getting-started/install.html){.external}
- [OpenStack executables](https://access.redhat.com/documentation/en-us/){.external}

> [!primary]
>
> This tutorial was created using **Terraform v0.9.11**.

## Instructions

### Creating the Terraform environment

After the Terraform installation, create a directory for all text files that will describe your infrastructure:

```console
mkdir test_terraform && cd test_terraform
```

Now we can create a Terraform environment with the following command. This will allow you to create and manage the evolution of your infrastructure.

```console
terraform env new test_terraform
```

### Creating resources

#### Creating the provider

In Terraform, you specify "providers" for your cloud environment. A "provider" (such as OVHcloud) hosts your OpenStack infrastructure resources. 

In a file named *provider.tf*, put the following lines:

```python
1. # Configure the OpenStack provider, hosted by OVHcloud
2. provider "openstack" {
3.   auth_url = "https://auth.cloud.ovh.net/v3" # Authentication URL
4.   domain_name = "default" # Domain name always "default" for OVHcloud
5.   alias = "ovh"
6. }
```

Right-click <a href="https://raw.githubusercontent.com/ovh/docs/develop/pages/platform/public-cloud/how_to_use_terraform/images/TF1.txt" download>here and "Save Link As"</a> to download the code-only text file.

The "alias" is a unique identifier for a provider. For example, if you have two OpenStack providers with different credentials, you must precise each provider in the resource.

#### Creating an instance

In Terraform, a "resource" is a component of your infrastructure. This can be an instance, a storage block, delivered by OpenStack provider or network delivered by OVHcloud provider.

To create an instance, you need at least:

- An instance name
- An image
- A flavor
- An SSH key

To find all available flavor names (instance types), you can enter the following command:

```console
nova flavor-list
```

The following examples will use the "s1-2" flavor.

To find all all available images, you can enter the following command:

```console
glance image-list
```

The following examples will use the "Ubuntu 16.04" image.

For example purposes, we will create a simple instance with Ubuntu 16.04 with the flavor s1-2 and then upload an SSH key. Put the following lines into a file named *simple_instance.tf*:

```python
1. # Create an SSH key pair resource
2. resource "openstack_compute_keypair_v2" "test_keypair" {
3.   provider = "openstack.ovh" # Provider name
4.   name = "test_keypair" # SSH key name
5.   public_key = "${file("~/.ssh/id_rsa.pub")}" # Path of your SSH key
6. }
7. 
8. # Create an instance
9. resource "openstack_compute_instance_v2" "test_terraform_instance" {
10.   name = "terraform_instance" # Instance name
11.   provider = "openstack.ovh" # Provider name
12.   image_name = "Ubuntu 16.04" # Image name
13.   flavor_name = "s1-2" # Flavor name
14.   # Get the name of the resource openstack_compute_keypair_v2 named test_keypair
15.   key_pair = "${openstack_compute_keypair_v2.test_keypair.name}"
16.   network {
17.     name = "Ext-Net" # Add the network component to reach your instance
18.   }
19. }
```

Right-click <a href="https://raw.githubusercontent.com/ovh/docs/develop/pages/platform/public-cloud/how_to_use_terraform/images/TF2.txt" download>here and "Save Link As"</a> to download the code-only text file.

With the configuration files created, you can enter the following command to create the instance and the SSH key pair resources:

```console
terraform apply
```

If you log in to your OVHcloud Control Panel now, you will find an instance named "terraform_instance".

Note that creating a second, identical instance with `terraform apply` will not work. Terraform applies change only if it recognises a difference in your configuration files or a new file.

#### Creating multiple instances

In this section, we will create an Ubuntu instance of the flavor "s1-2" in three different regions.

You can find all region names by checking this OVHcloud API endpoint:

> [!api]
>
> @api {GET} cloud/project/{serviceName}/region
>

We will use the following OVHcloud regions for this example:

- GRA3
- SBG3
- BHS3

You could create three resources named "openstack_compute_instance_v2" and change the region parameter for each. It can however become difficult to manage files with a large amount of identical resources.

A better method is to use the resource meta parameter called "count". It allows you to tell Terraform to create the same resource several times.

To do this, we will create a file named "multiple_instance.tf". In it, we first define a variable containing the three regions, and then add an instance creation counter:

```python
1. # Create a variable of the type list, containing OVHcloud regions
2. variable "region" {
3.   type = "list"
4.   default = ["GRA3", "SBG3", "BHS3"]
5. }
6. 
7. # Create SSH key
8. resource "openstack_compute_keypair_v2" "test_keypair_all" {
9.   count = "${length(var.region)}"
10.   provider = "openstack.ovh" # Provider name
11.   name = "test_keypair_all" # SSH key name
12.   public_key = "${file("~/.ssh/id_rsa.pub")}" # SSH key path
13.   region = "${element(var.region, count.index)}"
14. }
15. 
16. 
17. # Create a resource containing an OpenStack instance from each region
18. resource "openstack_compute_instance_v2" "instances_on_all_regions" {
19.   # Number of times the resource will be created
20.   # defined by the length of the list named region
21.   count = "${length(var.region)}"
22.   provider = "openstack.ovh" # Provider name
23.   name = "terraform_instances" # Name of your instance
24.   flavor_name = "s1-2" # Flavor name
25.   image_name = "Ubuntu 16.04" # Image name
26.   # element is a function calling an element at the position
27.   # count.index in the list var.region
28.   region = "${element(var.region, count.index)}"
29.   # Calling name variable from the resource openstack_compute_keypair_v2
30.   # named test_keypair
31.   key_pair = "${element(openstack_compute_keypair_v2.test_keypair_all.*.name, count.index)}"
32.   network {
33.     name = "Ext-Net" # Add the network component to reach your instance
34.   }
35. }
```

Right-click <a href="https://raw.githubusercontent.com/ovh/docs/develop/pages/platform/public-cloud/how_to_use_terraform/images/TF3.txt" download>here and "Save Link As"</a> to download the code-only text file.

Apply your changes with the following command:

```console
terraform apply
```

Terraform can create multiple instances with this method but you can use it to modify you current infrastructure as well.

#### Modifying an instance

In this example we will attach a new storage volume to our first instance. Open and edit the file named "simple_instance.tf", then add the following lines:

```python
1. # Create a resource for storage
2. resource "openstack_blockstorage_volume_v2" "volume_to_add" {
3.   name = "simple_volume" # Storage name
4.   size = 10 # Storage size
5.   provider = "openstack.ovh" # Provider name
6. }
7. 
8. # Attach the volume created previously to the instance
9. resource "openstack_compute_volume_attach_v2" "attached" {
10.   # Get the ID of your openstack_compute_instance_v2 named test_terraform_instance
11.   instance_id = "${openstack_compute_instance_v2.test_terraform_instance.id}"
12.   # Get the ID of your openstack_blockstorage_volume_v2 named volume_to_add
13.   volume_id = "${openstack_blockstorage_volume_v2.volume_to_add.id}"
14. }
```

Right-click <a href="https://raw.githubusercontent.com/ovh/docs/develop/pages/platform/public-cloud/how_to_use_terraform/images/TF4.txt" download>here and "Save Link As"</a> to download the code-only text file.

Apply your changes with the following command:

```console
terraform apply
```

#### Creating an instance in the OVHcloud network (vRack)

The Terraform OVHcloud plugin can manage private networks, private subnets, Public Cloud users and vRack attachments. In this part we will focus on the network creation.

First, the OVHcloud identifiers have to be initialised in your environment. Customise the commands with your actual values:

```console
export OVH_ENDPOINT=ovh-eu
export OVH_APPLICATION_KEY=Your_application_key_OVH(or_AK)
export OVH_APPLICATION_SECRET=Your_application_secret_key_OVH(or_AS)
export OVH_VRACK=vRack_identification_OVH
export OVH_PUBLIC_CLOUD=Your_tenant_id_openstack
export TF_ACC=1
export OVH_CONSUMER_KEY=Your_token(or_CK)
```

Now add the following lines to the file "provider.tf":

```python
1. # Configure the provider
2. provider "ovh" {
3.   endpoint = "ovh-eu" # Endpoint of the provider
4.   alias = "ovh" # Provider alias
5. }
```

Right-click <a href="https://raw.githubusercontent.com/ovh/docs/develop/pages/platform/public-cloud/how_to_use_terraform/images/TF5.txt" download>here and "Save Link As"</a> to download the code-only text file.

Create a file "create_private_instance.tf" and enter the following:

```python
1. # Create a private network
2. resource "ovh_publiccloud_private_network" "network" {
3.    project_id = "OS_TENANT_ID" # With OS_TENANT_ID your tenant id's project
4.    name = "private_network" # Network name
5.    regions = ["OS_REGION_NAME"] # With OS_REGION_NAME the OS_REGION_NAME environment variable
6.    provider = "ovh.ovh" # Provider name
7.    vlan_id = 168 # vRack identifier
8. }
9. 
10. # Create a subnet in the previous network
11. resource "ovh_publiccloud_private_network_subnet" "subnet" {
12.    project_id = "OS_TENANT_ID" # With OS_TENANT_ID your tenant ID's project
13.    # Get the ID of the resource ovh_publiccloud_private_network named network
14.    network_id = "${ovh_publiccloud_private_network.network.id}"
15.    start = "192.168.168.100" # First IP for the subnet
16.    end = "192.168.168.200" # Last IP for the subnet
17.    network = "192.168.168.0/24" # Global network
18.    dhcp = true # Active the DHCP
19.    region = "OS_REGION_NAME" # With the OS_REGION_NAME the OS_REGION_NAME environment variable
20.    provider = "ovh.ovh" # Provider name
21. }
22. 
23. # Create an instance with 2 network interfaces
24. resource "openstack_compute_instance_v2" "proxy_instance" {
25.   provider = "openstack.ovh" # Provider name
26.   name = "proxy_instance" # Instance name
27.   image_name = "Ubuntu 16.04" # Image name
28.   flavor_name = "s1-2" # Flavor name
29.   # Get the name of the resource openstack_compute_keypair_v2 named test_keypair
30.   key_pair = "${openstack_compute_keypair_v2.test_keypair.name}"
31.   # Add to the public network and the private network
32.   network = [{name = "Ext-Net"}, {name = "${ovh_publiccloud_private_network.network.name}"}]
33. }
```

Right-click <a href="https://raw.githubusercontent.com/ovh/docs/develop/pages/platform/public-cloud/how_to_use_terraform/images/TF6.txt" download>here and "Save Link As"</a> to download the code-only text file.

With this, we have added a new instance to your Public Cloud project and enabled a private and public network interface on it.

#### Creating an infrastructure for a web site

In this example, we will create a basic web site infrastructure using Terraform and the OVHcloud private network. The components created are:

- a private network
- a subnet
- two instances with two network interfaces each: the first one public and the second one private
- an instance with a private interface and two additional disks

![public-cloud](images/basic_infrastructure_for_a_web_site.png){.thumbnail}

Create a file "named simple_web_site.tf" and enter the following lines:

```python
1. # Create a private network
2. resource "ovh_publiccloud_private_network" "private_network" {
3.   name = "backend" # Network name
4.   regions = ["OS_REGION_NAME"] # With the OS_REGION_NAME the OS_REGION_NAME environment variable
5.   provider = "ovh.ovh" # Provider name
6.   project_id = "OS_TENANT_ID" # With OS_TENANT_ID your tenant ID's project
7.   vlan_id = 42 # vRack identifier
8. }
9. 
10. # Create a private subnet
11. resource "ovh_publiccloud_private_network_subnet" "private_subnet" {
12.   # Get the ID of the resource ovh_publiccloud_private_network named
13.   # private_network
14.   network_id = "${ovh_publiccloud_private_network.private_network.id}"
15.   project_id = "OS_TENANT_ID" # With OS_TENANT_ID your tenant ID's project
16.   region = "OS_REGION_NAME" # With OS_REGION_NAME the OS_REGION_NAME environment variable
17.   network = "192.168.42.0/24" # Global network
18.   start = "192.168.42.2" # First IP for the subnet
19.   end = "192.168.42.200" # Last IP for the subnet
20.   dhcp = false # Deactivate the DHCP service
21.   provider = "ovh.ovh" # Provider name
22. }
23. 
24. # Find the most recent image Archlinux
25. data "openstack_images_image_v2" "archlinux" {
26.   name = "Archlinux" # Image name
27.   most_recent = true # Find the most recent
28.   provider = "openstack.ovh" # Provider name
29. }
30. 
31. # List of private IPs for the instance
32. variable "front_private_ip" {
33.   type = "list"
34.   default = ["192.168.42.2", "192.168.42.3"]
35. }
36. 
37. # Create 2 instances with 2 network interfaces
38. resource "openstack_compute_instance_v2" "front" {
39.   # Number of times the instance will be created
40.   count = "${length(var.front_private_ip)}"
41.   provider = "openstack.ovh" # Provider name
42.   name = "front" # Instance name
43.   flavor_name = "s1-2" # Flavor name
44.   image_id = "${data.openstack_images_image_v2.archlinux.id}" # Image ID
45.   security_groups = ["default"] # Add into a security group
46.   network = [
47.     {
48.       name = "Ext-Net" # Public interface name
49.     },
50.     {
51.       # Private interface name
52.       name = "${ovh_publiccloud_private_network.private_network.name}"
53.       # Give an IP address depending on count.index
54.       fixed_ip_v4 = "${element(var.front_private_ip, count.index)}"
55.     }
56.   ]
57. }
58. 
59. # Create 1 hard disk attachable and detachable
60. resource "openstack_blockstorage_volume_v2" "backup" {
61.   name = "backup_disk" # Hard disk name
62.   size = 10 # Size
63.   provider = "openstack.ovh" # Provider name
64. }
65. 
66. # Create 1 instance with 1 network interface and 1 extra hard disk
67. resource "openstack_compute_instance_v2" "back" {
68.   provider = "openstack.ovh" # Provider name
69.   name = "back" # Instance name
70.   flavor_name = "s1-2" # Flavor name
71.   image_id = "${data.openstack_images_image_v2.archlinux.id}" # Image ID
72.   security_groups = ["default"] # Add into a security group
73.   network = [
74.     {
75.       name = "${ovh_publiccloud_private_network.private_network.name}" # Private interface name
76.       fixed_ip_v4 = "192.168.42.150" # Private IP
77.     }
78.   ]
79.   # Instance hard disk bootable content the Operating System
80.   block_device {
81.     uuid = "${data.openstack_images_image_v2.archlinux.id}" # Image ID
82.     source_type = "image" # Source type of the block device
83.     destination_type = "local" # Destination
84.     volume_size = 10 # Size
85.     boot_index = 0 # Boot order
86.     delete_on_termination = true # Whether it will be deleted when the instance is deleted
87.   }
88.   # Hard disk create and attach directly to the instance
89.   block_device {
90.     source_type = "blank" # Source type of the block device
91.     destination_type = "volume" # Destination
92.     volume_size = 20 # Size
93.     boot_index = 1 # Boot order
94.     delete_on_termination = true # Whether it will be deleted when the instance is deleted
95.   }
96.   # Hard disk created previously and attached
97.   block_device {
98.     uuid = "${openstack_blockstorage_volume_v2.backup.id}" # Volume ID
99.     source_type = "volume" # Source type of the block device
100.     destination_type = "volume" # Destination
101.     boot_index = 2 # Boot order
102.     delete_on_termination = true # Whether it will be deleted when the instance is deleted
103.   }
104. }
```

Right-click <a href="https://raw.githubusercontent.com/ovh/docs/develop/pages/platform/public-cloud/how_to_use_terraform/images/TF7.txt" download>here and "Save Link As"</a> to download the code-only text file.

Apply your changes with the following command:

```console
terraform apply
```

### Deleting an infrastructure

To remove every resource you can enter the following command:

```console
terraform destroy
```

## Go further

Join our community of users on <https://community.ovh.com/en/>.
