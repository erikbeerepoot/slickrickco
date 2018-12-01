---
layout: post
title:  "Automating the creation and configuration of your AWS Deep Learning instance"
date:   2018-11-30 06:07:00 -0700
categories: machine-learning aws ec2 deep-learning
use_math: true
--- 

### Introduction 

One important aspect of software development is the ability to do one-step builds. One-step builds are builds that deploy your application along with all 
the necessary infrastructure without user intervention, other than the initial action taken to start the process. This confers some important benefits:
- Repeatability: Your application environment is always the same, which means your application has the necessary components to run, and you have a known state 
  you can use a start for debugging.
- Ease of testing: You can easily deploy a new instance of your service or application in a staging environment for testing.
- Ease of deployment: The deployment process is simple and leaves little room for error.
- Bootstrapping latency: It's quick to get up and running - even a complex deployment process is going to faster when steps don't have to be taken manually
- Completeness: Manually configuring your environment invites the tendency to cut corners; since every step takes a bit longer, you can speed up the process by
  leaving some steps out.

In this post, we're going to automate the creation & configuration of an EC2 deep learning instance, with the following end state:
- Have a running P2 deep learning EC2 instance
- A configured Jupyter notebook server running in the background
- Jupyter plugin configurator & a set of base plugins installed
- A specific (user chosen) folder copied to the remote instance 

To accomplish this, we will be using two tools:
- [Terraform](https://www.terraform.io)
- [Ansible](https://www.ansible.com/)

Terraform is used to create the relevant AWS infrastructure, which in our case will mean the EC2 instance configuration. Ansible is used for configuration 
management; we will use it to install the relevant softare, copy files and launch the jupyter notebook server. Let's get started!

### Getting terraform and ansible

First, we'll need to install Terraform and Ansible:

{% highlight bash %}

brew install terraform
pip install ansible 

{% endhighlight %}

### Creating the terraform configuration 
It's good security practice to [create an IAM user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html) to limit the scope of your credentials in AWS. 
Follow the necessary steps to accomplis that, and take note of the Access ID and Access Key. Alternatively, you can have Terraform rely on your credentials in `~/.aws/credentials`.
First, define the provider block:

{% highlight yaml  %}

# Configure the AWS Provider

provider "aws" {
  # Access key and secret key are optional, if you've got your credentials defined in ~/.aws/credentials
  access_key = "${var.aws_access_key}"
  secret_key = "${var.aws_secret_key}"
  region     = "us-west-2"
}
{% endhighlight %}

Next, let's define some security rules in a security group:

{% highlight yaml  %}
{% raw %}

resource "aws_security_group" "allow_ssh" {
  name        = "allow_ssh"
  description = "allows ingress of ssh traffic"

  ingress {
    from_port = 22
    to_port   = 22
    protocol  = "tcp"
    # Allow access from anywhere - you can restrict this to specific IPs (recommended)
    cidr_blocks = ["0.0.0.0/0"]
  }
   
   # ephemeral ingress ports are required for updates / pip
   ingress {
    from_port = 32768
    to_port   = 65535
    protocol  = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port = 0
    to_port   = 0
    protocol  = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
{% endraw %}
{% endhighlight %}

Here, we're allowing `ssh` ingress, since we can easily tunnel Jupyter over ssh. Furthermore, the ansible provisioner will also leverage ssh to peform the configuration changes. We also allow
ingress on the ephemeral ports, so we can use pip/conda/apt-get.

Now, we create a configuration template for the deep learning EC2 instance we're going to create:

{% highlight yaml  %}
resource "aws_instance" "deep-learning" {
  ami           = "ami-0688c8f24f1c0e235"
  instance_type = "p3.2xlarge"
  security_groups = ["allow_ssh"]
}
{% endhighlight %}

We've chosen the `p3.2xlarge` instance type here, but you can change this to an instance type of your choosing. Note the referencing of the `allow_ssh` security group, this is necessary 
to configure the instance to use it.

Next, run:

{% highlight bash  %}
terraform init
terraform apply
{% endhighlight %}


At this point, you should have a running EC2 instance with ssh access!

### Automatic provisioning using Ansible


While we now have a running EC2 instance of the appropriate type, we still need to configure the Jupyter server, copy any files we need and do any other configuration we might want.

First, let's create an inventory file:

{% highlight bash %}
[learning-boxes:vars]
ansible_user=ubuntu

[learning-boxes]
ec2-34-219-62-211.us-west-2.compute.amazonaws.com
{% endhighlight %}

We also need to create a configuration file to point ansible to the right ssh key:
{% highlight bash %}
[defaults]
ansible_ssh_private_key_file = "/path/to/your/key.pem"
{% endhighlight %}


All that's left to do now is to configure the ec2 instance:
{% highlight yaml  %}
{% raw %}
---
- hosts: all
  vars:
        jupyter_json_config: ~/.jupyter/jupyter_notebook_config.json
        jupyter_py_config: ~/.jupyter/jupyter_notebook_config.py
        virtualenv: tensorflow_p36
        copy_dir: ~/Dropbox/Code/machine-learning/
  tasks:
      	- name: Get running processes
          shell: "ps -ef | grep -v grep | grep -w jupyter | awk '{print $2}'"
          register: running_processes

        - name: Kill running processes
          shell: "kill {{ item }}"
          with_items: "{{ running_processes.stdout_lines }}"

        - wait_for:
                path: "/proc/{{ item }}/status"
                state: absent
          with_items: "{{ running_processes.stdout_lines }}"
          ignore_errors: yes
          register: killed_processes

        - name: Force kill stuck processes
          shell: "kill -9 {{ item }}"
          with_items: "{{ killed_processes.results | select('failed') | map(attribute='item') | list }}"

        - name: Create cert directory
          file:
                path: ~/ssl/
                state: directory

        - name: Create ssl cert for Jupyter
          command: openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout "ssl/cert.key" -out "ssl/cert.pem" -batch

        - name: Generate Jupyter configuration file
          expect:
                command: jupyter notebook --generate-config
                responses:
                        (?i)default config?: "y"

        - name: Set Jupyter password
          expect:
                command: jupyter notebook password
                responses:
                      (?i)password: "MySekretPa$$word"

        - name: Slurp Jupyter password file
          slurp:
                src: "{{ jupyter_json_config }}"
          register: jupyter_json_config

        - debug:
                msg: "{{jupyter_json_config['content'] | b64decode }}"

        - name: Set hashed password
          set_fact:
                password_hash: "{{ (jupyter_json_config['content'] | b64decode | from_json)['NotebookApp']['password'] }}"

        - name: Configure Jupyter to use keys, prevent window opening, and set bind address
          blockinfile:
                path: "{{ jupyter_py_config }}"
                block: |
                      c = get_config()  # Get the config object.
                      c.NotebookApp.certfile = u'/home/ubuntu/ssl/cert.pem' # path to the certificate we generated
                      c.NotebookApp.keyfile = u'/home/ubuntu/ssl/cert.key' # path to the certificate key we generated
                      c.NotebookApp.ip = '*'  # Serve notebooks locally.
                      c.NotebookApp.open_browser = False  # Do not open a browser window by default when using notebooks.

        - name: Put pashword hash in Jupyter config file
          lineinfile:
                path: "{{ jupyter_py_config }}"
                regexp: '^c\.NotebookApp\.password'
                line: 'c.NotebookApp.password = "{{password_hash}}"'
    
        - name: Install jupyter plugins
          shell: pip3 install jupyter_contrib_nbextensions --user

        - name: Install nbextensions javascript and css files
          shell: |
                  source activate {{ virtualenv }}
                  jupyter contrib nbextension install --user
          args:
                executable: /bin/bash

        - name: Start Jupyter notebook server
          shell: |
                  source activate {{ virtualenv }}
                  jupyter notebook </dev/null >/dev/null 2>&1 &
          args:
                executable: /bin/bash
          async: 2592000
          poll: 0

        - name: Copy files to remote
          copy:
                src: "{{ copy_dir }}"
                dest: ~/workspace/
{% endraw %}                
{% endhighlight %}

These steps are pretty straightforward:

- Kill all Jupyter processes that are running.
- Generate ssl cert & password, and configure Jupyter to use it.
- Install jupyter nbextensions + some plugins.
- Start jupyter.
- Copy desired files.

Finally, a little script to tie it all together:

{% highlight bash %}
#!/bin/bash
# Script to automatically configure a new ec2 box end-to-end

# Setup new ec2 instance (or change existing)
terraform apply -auto-approve

# Delete existing hosts from inventory
sed '/ec2.*\.compute\.amazonaws\.com/d' inventory.yaml > temp.inventory 
mv temp.inventory inventory.yaml 

# Add host from terraform 
terraform show | grep public_dns | awk '/public_dns =/ { print $3 }' >> inventory.yaml

# Configure ec2 instance
ansible-playbook -i inventory.yaml configure.yml
{% endhighlight %}

It's all pretty simple, but this can save us a lot of time. Find the complete source code [here](https://github.com/erikbeerepoot/machine-learning/tree/master/inf).
