Hello Folks,

Ansible is a well-known tool for configuration management with its own advantages. We can use ansible for Creating AWS resources too.

Sounds great, right?

In this example, we are setting up a basic AWS infrastructure and launching a website from the GitHub repository.

![setup-git+ansible](https://user-images.githubusercontent.com/61390678/221956129-a2bb67c7-3785-40ba-b12b-a41780b6cc25.png)


Before proceeding you have to make sure that you have installed the latest version of ansible and required python modules such as boto, botocore, boto3.

If it is not installed, don’t worry. You can use the following commands to install the latest version of ansible and python modules.

```
$ yum install python3 python3-pip
$ pip3 install ansible
```

Make sure that the ansible installation was successful.

```
$ ansible --version

ansible [core 2.11.12]
  config file = None
  configured module search path = ['/home/ec2-user/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /home/ec2-user/.local/lib/python3.7/site-packages/ansible
  ansible collection location = /home/ec2-user/.ansible/collections:/usr/share/ansible/collections
  executable location = /home/ec2-user/.local/bin/ansible
  python version = 3.7.16 (default, Dec 15 2022, 23:24:54) [GCC 7.3.1 20180712 (Red Hat 7.3.1-15)]
  jinja version = 3.1.2
  libyaml = True
  ```
  
  Now we can install the required modules using pip.

```
pip3 install boto boto3 botocore
```

Also, we need some additional ansible roles for our code, so we are installing the same using ansible-galaxy.

```
ansible-galaxy collection install amazon.aws
```

Now Let’s review the code, It consists of 2 plays. The second play will be running on a dynamic inventory created by the 1st play.

 Now let’s have a deeper look into our code [here](https://github.com/antony-a-n/Launching-AWS-resources-using-Ansible-and-deploying-a-website-from-Git/blob/902367263dfeb82769c88bd72c024386570522e1/main.yml).
 
 before executing please make sure that you have created an IAM user with the required permissions.

We have configured the access_key and secret_key for the API request on a file named login.yml.Also, we have set the region, project &env as variables.

In the 1st task, we are creating a key pair required for launching our webserver.

Once it is created we are saving the private key with a .pem extension.

After creating the keypair, we need to set up a security group. In this example, we are creating a security group with ports 80,22, and 443 open for ingress. By default, all the egress ports are open.

Now we can set up our instances, and for that, you have to specify the VPC, region, instance type, Key, security group, AMI id, etc.

We are launching 3 instances here by using exact_count. You can use normal count here, but the issue is that whenever you run the playbook, it will launch new instances based on the number you have mentioned with the count.

This can be avoided using exact_count, it is working in hand with filters, we are creating new instances based on a few tags, Whenever you run the playbook again, it will 1st check the no of instances with the mentioned tag. If the no of instances matched the no of counts you have provided with exact_count. If they are the same, it won’t launch a new instance.

Once you have launched the instances, we have to fetch the details of the newly created instances for setting up the dynamic inventory.

Then we are filtering the instances based on the tag, we are provided with and register the details of instances.

After fetching all the details, as the last task we are creating a dynamic inventory for running our second play and it includes the ansible_user, port, key file, etc.

Now we can execute the 2nd play, the group will be the members of the dynamic inventory.

In the second play we are launching our website, and for that, we have to specify some variables such as git repo URL, HTTP port, HTTP user, group, etc.

We need to install a few packages for our setup. Including git, httpd & PHP.

Once the packages are installed and running, we can finish the rest of our tasks such as changing permissions and configuring httpd.conf, virtualhost file etc.

We are using template format here.Once the required files are reached the destination server, we can create the document root for cloning our website contents.

Now we can clone the contents using the git module, from the location, we can move the contents to the document root.

Since we have setup handlers to reload and restart apache, whenever there is a change in the tasks, the handlers will trigger.

Also we are executing the playbook in serial mode, so it will perform tasks on a single machine at a time.

Once the task fully executed, you will be able to access the website .

Use the following command to run the playbook.

```
ansible-playbook main.yml
```

Please have a check and Let me know if you are facing any issues.

Thank you for reading :)
