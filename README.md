Role Name
=========

> The Cornerstone is the first stone set in the construction of a masonry foundation, important since all other stones will be set in reference to this stone, thus determining the position of the entire structure.

Cornerstone is a simple ansible role to create a single Virtual Machine on supported Cloud providers.

Requirements
------------

As Cornerstone creates resources within Cloud Providers, it depends on appropriate SDKs for each provider to be installed.

Role Variables
--------------

Available variables are listed below, along with default values (see defaults/main.yml):

    # Platform you wish to deploy a Cornerstone VM onto
    # Currently supports aws, azure, libvirt
    cornerstone_platform: aws

    # Prefix - What prefix for objects to use. Eg myenv, myapp, etc
    cornerstone_prefix: cornerstone
   
    # Tags - Tags that can be used.
    cornerstone_tag_purpose: "Openshift"
    cornerstone_tag_role: "Master"    
 
    # Cornerstone VM Name
    cornerstone_vm_name: "{{ cornerstone_prefix }}-vm"

    # Determines if a vm should be added or removed. 
    # This works for AWS and libvirt but not Azure yet.
    # Accepted values are present and absent only.
    # Default value is present
    cornerstone_vm_state: present

    # SSH Credentials
    cornerstone_ssh_admin_username: rhadmin
    cornerstone_ssh_admin_pubkey: <add_your_ssh_public_key_data>
    # The AWS SSH Key to use. Must be pre-created in AWS
    cornerstone_aws_ssh_key_name: mykey
    cornerstone_ssh_user: ec2-user
    cornerstone_ssh_key_path: "/home/user/.ssh/userssh.pem" 
    # Extra args added for ssh configuration, default value adds
    # StrictHostKeyChecking=no
    cornerstone_ssh_extra_args: "-o StrictHostKeyChecking=no"

 
    # API Credentials
    # What Boto profile to use for AWS
    cornerstone_aws_profile: myprofile

    # Platform Location/Region
    # Which location/region do you wish to use?
    # Currently supports short versions of aws/azure regions eg uksouth (azure), eu-west-2 (aws)
    cornerstone_location: uksouth

    # Azure Resource Group
    cornerstone_azure_resource_group: "{{ cornerstone_prefix }}-rg"

    # Common Network Objects
    cornerstone_virtual_network_name: "{{ cornerstone_prefix }}vnet"
    cornerstone_virtual_network_cidr: "10.1.0.0/16"
    cornerstone_subnet_name: "{{ cornerstone_prefix }}subnet"

    # Azure Specific Network Objects
    cornerstone_azure_subnet_cidr: "10.1.0.0/20"

    # AWS Specific Network Objects
    cornerstone_aws_subnet_cidr_list:
      - "10.1.0.0/20"
      - "10.1.16.0/20"
      - "10.1.32.0/20"

    # Private or Public IP
    cornerstone_vm_assign_public_ip: true
    cornerstone_vm_public_ip: <elastic ip precreated>
    cornerstone_public_private_ip: public
    cornerstone_publicip_allocation_method: Dynamic
    cornerstone_publicip_domain_name: null

    # Cornerstone Rules to maintain, this is the list that will 
    # determine which rules are enabled for the security group.
    cornerstone_sg:
      - name: "cs-sg"
	description: Security group for aws
	region: "{{ cornerstone_location }}"
	rules:
	  - proto: tcp
	    from_port: 22
	    to_port: 22
	    group_name: ""
	    cidr_ip: 0.0.0.0/0
	    rule_desc: "allowSSHin_all"
	  - proto: tcp
	    from_port: 443
	    to_port: 443
	    group_name: ""
	    cidr_ip: 0.0.0.0/0
	    rule_desc: "allowHttpsin_all"
	  - proto: all
	    from_port: ""
	    to_port: ""
	    group_name: "cs-sg"
	    cidr_ip: 0.0.0.0/0
	    rule_desc: "allowAllfromSelf"

    # Virtual Machine
    cornerstone_vm_flavour: Standard_D2s_v3
    # AWS AMI Image ID
    cornerstone_vm_aws_ami: ami-12345678
    # Azure Image
    cornerstone_vm_image_offer: RHEL
    cornerstone_vm_image_publisher: RedHat
    cornerstone_vm_image_sku: 7-RAW
    cornerstone_vm_image_version: latest

    # Security group to use, if not specified, default will be used.
    cornerstone_vm_aws_sg: 

    # Disks
    # Type of managed disk (AWS)
    cornerstone_aws_vm_disk_managed: gp2
    # Type of managed disk (Azure)
    cornerstone_azure_vm_disk_managed: Standard_LRS
    # Base OS Disk Size (Standard AWS base is 10GB, Azure is 32GB)
    cornerstone_vm_os_disk_size: 32
    # Do we want a separate data disk?
    cornerstone_vm_data_disk: false
    
    # How large?
    #### IMPORTANT #####################################################
    # AWS ec2_instance module for ansible 2.7+ currently has a problem #
    # with converting strings to ints for disk size.                   #
    # To resolve add the code mentioned in                             #
    # https://github.com/ansible/ansible/pull/55716/files              #
    ####################################################################

    cornerstone_vm_data_disk_size: 64

    # Libvirt specific variables
    
    # Where are vms stored, default location is /var/lib/libvirt/images/
    cornerstone_vm_location: /var/lib/libvirt/images/

    # A working temp directory on the libvirt host where files can be created and deleted.
    # Default value is /tmp
    cornerstone_working_dir: /tmp/

    # Name of the template to use to spin up linux vms.
    # Default value is rhel7-image
    cornerstone_vm_libvirt_template: rhel7-image

    # Name of the vm file type that will be used.
    # Default value is qcow2
    cornerstone_vm_libvirt_file_type: qcow2

    # The type of vm that will be created. Only supports "emptyvm" at this stage.
    # Default value is no value
    cornerstone_vm_libvirt_vmtype: 

    # The operating system of the virtual machine
    # Default value is linux, look at libvirt documentation for more values.
    cornerstone_vm_libvirt_vmos: linux

    # VM resources to allocate
    # Memory value MUST be in MB
    cornerstone_vm_libvirt_vmmem:
    cornerstone_vm_libvirt_vmcores:

    # VM networking variables
    cornerstone_vm_libvirt_ip:
    cornerstone_vm_libvirt_subnet:
    cornerstone_public_private_gateway:
    cornerstone_public_private_dns1:
    cornerstone_public_private_dns2:


Dependencies
------------

Not a dependency per-se, but used alongside Finial, can subscribe RHEL systems to the Red Hat Network.

For AWS deployments the aws_cli is required. Role for this can be found at https://github.com/kenhitchcock/ansible-role-awscli

Example Playbook
----------------

# For and AWS deployment this will work for multiple instances.
    - hosts: localhost
      vars:
        cornerstone_sg:
	  - name: "cs-sg"
	    description: Security group for aws
	    region: "{{ cornerstone_location }}"
	    rules:
	      - proto: tcp
	        from_port: 22
	        to_port: 22
	        group_name: ""
	        cidr_ip: 0.0.0.0/0
	        rule_desc: "allowSSHin_all"
	      - proto: tcp
	        from_port: 443
	        to_port: 443
	        group_name: ""
	        cidr_ip: 0.0.0.0/0
	        rule_desc: "allowHttpsin_all"
	      - proto: all
	        from_port: ""
	        to_port: ""
	        group_name: "cs-sg"
	        cidr_ip: 0.0.0.0/0
	        rule_desc: "allowAllfromSelf"
        guests:
          aws-instance01:
            cornerstone_platform: aws
            cornerstone_tag_purpose: "Openshift"
            cornerstone_tag_role: "Master"
            cornerstone_vm_name: aws-instance01
            cornerstone_vm_state: present
            cornerstone_location: eu-west-2
            cornerstone_vm_aws_az: eu-west-2a
            cornerstone_vm_flavour: t2.micro
            cornerstone_vm_aws_ami: ami-051fb39c3a16c8a85
            cornerstone_vm_aws_sg: "cs-sg"
            cornerstone_virtual_network_name: "{{ cornerstone_prefix }}vnet"
            cornerstone_virtual_network_cidr: "10.1.0.0/16"
            cornerstone_subnet_name: "{{ cornerstone_prefix }}subnet"
            cornerstone_public_private_ip: public
            cornerstone_vm_assign_public_ip: true
            cornerstone_vm_public_ip: 3.9.8.123
            cornerstone_publicip_allocation_method: Dynamic
            cornerstone_publicip_domain_name: null
            cornerstone_vm_data_disk: true
            cornerstone_vm_data_disk_device_name: "/dev/sdb"
            cornerstone_vm_data_disk_size: "15"
          aws-instance02:
            cornerstone_platform: aws
            cornerstone_tag_purpose: "Openshift"
            cornerstone_tag_role: "Master"
            cornerstone_vm_name: aws-instance02
            cornerstone_vm_state: present
            cornerstone_location: eu-west-2
            cornerstone_vm_aws_az: eu-west-2b
            cornerstone_vm_flavour: t2.micro
            cornerstone_vm_aws_ami: ami-051fb39c3a16c8a85
            cornerstone_vm_aws_sg: "cs-sg"
            cornerstone_virtual_network_name: "{{ cornerstone_prefix }}vnet"
            cornerstone_virtual_network_cidr: "10.1.0.0/16"
            cornerstone_subnet_name: "{{ cornerstone_prefix }}subnet"
            cornerstone_public_private_ip: public
            cornerstone_vm_assign_public_ip: true
            cornerstone_vm_public_ip: 3.9.8.124
            cornerstone_publicip_allocation_method: Dynamic
            cornerstone_publicip_domain_name: null
            cornerstone_vm_data_disk: true
            cornerstone_vm_data_disk_device_name: "/dev/sdb"
            cornerstone_vm_data_disk_size: "15"

      roles:
         - role: danhawker.cornerstone

# For Azure you cannot use the guest layout yet. The task will only create one instance at a time.

Future Releases
---------------

 - Add support for Libvirt
 - Add support for VSphere
 - Add support for RHEV
 - Add support for OpenStack

License
-------

MIT

Author Information
------------------

Dan Hawker [Github](https://github.com/danhawker)
Ken Hitchcock - Contributor
