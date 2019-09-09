# Exercise 2: Windows Domain with Linux Member(s)
## Table of Contents
- [Introduction](#introduction)
- [Goal](#goal)
- [Permissions](#permissions)
- [Test Deploy a VM](#test-deploy-a-vm)
- [Create Machine Credential](#Create-Machine-Credential)
- [Create an Inventory](#Create-an-Inventory)
- [Create Job Templates](#Create-Job-Templates)
- [Build Surveys](#Build-Surveys)
- [Finalize workflow template](#Finalize-workflow-template)
- [Execute the workflow](#Execute-the-workflow)
- [Cleanup](#Cleanup)

## Introduction
Welcome to exercise **2** of the Ansible for Linux & Windows Hands-on Lab!

The url of the **Ansible Tower** cluster where you work on is: https://tower-{guid}.rhpds.opentlc.com
Where {guid} is the 4 character code you received at the start of the LAB.

You have the following accounts available on your Tower server:

- **windowsadmin:**
This persona maintains all windows playbooks. He has admin rights on the Tower project for Windows playbooks. If you want to use his playbooks, you must ask him nicely.

- **linuxadmin:**
This persona maintains all linux playbooks. He has admin rights on the Tower project for Linux playbooks. If you want to use his playbooks, you must ask him even more nicely.

- **webadmin:**
This persona’s job is to build and maintain web stacks. He needs the playbooks from the other persona’s to do her or his job.

- **admin:**
The tower administrator maintains the Azure playbooks. He is a nice guy by default. He already gave permission to use them, including the credentials you need to be able to use them. Neat!

- **user:**
This is the persona who is able to run but not maintain workflows.

These accounts are all member of the Ansible Tower Organization _“ACME Corporation”_


The url of the **gitlab** where your repositories live is: https://control-{guid}.rhpds.opentlc.com
Where {guid} is the 4 character code you received at the start of the LAB.

You have the following accounts available on your Git server:

- **git:**
This account gives you write access to all repositories, so if you feel like customizing the playbook, be my guest ;-)

All these accounts have password: _r3dh4t1!_ (that includes the !)

> **_Note:_**
>
> A trusted certificate is not installed, so you need to accept an exception for this in your browser.
> This lab uses Azure. When creating resources in Azure there are restrictions on VM names, usernames and passwords. Read [here](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/faq#are-there-any-computer-name-requirements) before you start if you do not know these restrictions!

[(top)](#table-of-contents)

## Goal
Here is (again) an example of what kind of workflow you are tasked to build:
![example workflow](https://github.com/fvzwieten/media/blob/master/exercise2.png "Example Workflow")
Remember, this is just an example!

You are tasked to build this workflow as the user “linuxadmin” with the added ability to have user “user” to execute it as well.  

Summary of tasks:
* You create one Windows and one Linux VM in Azure
* You sync the inventory to import the newly created VM’s in Azure into the inventory
* You create a new AD domain on the Windows machine
* You create a Domain Admin user and a Normal user
* You join the Linux machine to the AD domain
* You log into the Linux machine with either SSH or Web Console using the Windows accounts in the Active Directory.

> **_Note:_**
>
>If you use Web Console, a trusted certificate is not installed, so you need to accept an exception for this in your browser.

[(top)](#table-of-contents)

## Permissions

User _linuxadmin_ currently only has access to it's own Linux project and the Azure project & job templates as shared by the Ansible Tower Global Admin. So, you need to grant access to the Windows project to be able to use it's playbooks:

* Log into Ansible Tower with the _windowsadmin_ account
* Go to _Projects_ in the left main menu. You see the projects this account has access to.
* Select _Windows_
* On the projects details page, select _PERMISSIONS_ and click the green + button.
* Select _linuxadmin_
* In the lower menu, choose _Use_ as the role.
* Click Save  
* Log out of Tower. That is the _switch_ icon in the upper right corner of the Tower UI.

> **_Note:_**
>
> If you want to assign roles to _teams_ (e.g. Web Team), you need to be a Tower admin.

[(top)](#table-of-contents)

## Test Deploy a VM

Nothing stops us from already deploying a VM in Azure. You can use that VM in the later workflow. So, let’s execute a playbook that will do just that and see how that works:
1. Log into Ansible Tower as _linuxadmin_
2. Check in _Projects_ if you now also see the _Windows_ project based on the permissions you assigned earlier.
3. Go to _Templates_ in the left main menu. Here, you see a list of job templates that begin with azure_ and are shared with you and you are allowed to execute.
4. Next to each template you see a _run_ icon in the form of a flying rocket. Click on the one next to the job template called _azure_create_vm_.
5. You will be presented with a _survey_. A survey is a simple form to ask for input parameters for this run. Each field is self explanatory. The password field is already prefilled with the password you also use to log into tower and git. You are free to change it, but then please remember it for later.
6. Fill all fields and press _NEXT_. Create a Linux webserver and choose values that you can later use in your workflow. Remember all input you use here. You need them later! Also make sure you enter valid values. Now click _NEXT_.
7. You get to (p)review what you entered. You also see the var names that will be populated with these values in the playbooks. Press _LAUNCH_ when all is good.
8. You now see the _JOBS_ window appear with logging in the right pane as the job executes. The left pane shows all metadata of the job.

> **_Notes:_**
>
>* You can safely ignore the purple warning message at the beginning of the logging.
>* The job will take some time to execute. A Windows VM deployment will typically take longer than a Linux VM (on Azure). You can continue with the next exercise or take a look at the playbooks in your git server while you wait and go back to this job at any time by going to the _My View_ main menu item and look up this job’s details. As you can see each Job has been given a unique number. Select _My Jobs_ to only see your own jobs.
>* When the job completes it will report in the logging the public IP address of the VM. You can log into this VM with either RDP or SSH (depending on the OS you’ve chosen) and the username/password you entered in the survey.
>* If, for some reason you want to remove this VM, you can execute the job template called _azure_delete_vm_. It will ask for the VM Name and will delete that VM from Azure.
>* Have a look at the playbooks in the _ansible4azure_ repository in your git server. They showcase how you can use Ansible with Azure.
>* If the browser window is too small, the interface will automatically switch to mobile view. Left and Right become Up and Down.
>* You have write access to the git repositories. Feel free to customize the playbooks to your own liking. Remember though that this server will be deleted after the workshop..

[(top)](#table-of-contents)

## Create Machine Credential

When you deploy machines, you needed to specify credentials to be able to log in to it. You also need credentials to be able to run Ansible playbooks _in_ the VM’s OS. Although the Windows admin and/or Linux admin could share their credentials to do that, that would mean you can log into their other machines as well. Better to create your own machine credential then! Here we go:
1. Log into Ansible Tower as _linuxadmin_ (if you haven’t already)
2. Go to _Credentials_ in the left main menu. You see some credentials already exist in your account:
   * There is an _Azure_ credential, given to you to be able to authenticate against Azure to be able to manage Azure resources, like VM’s
   * There is an SCM credential called "GiTea" to log into GiTea to sync the repositories.
   * There are also two _Vault_ credentials. What are those? A vault in Ansible is a file that has been encrypted using the ansible-vault tool. You can encrypt any file with this. In this Lab we encrypted 2 text files which hold credentials. One for accessing the database for write, the other one for configuring the access to the haproxy stats webpage. The encrypted files are protected with a password. We defined vault credentials in Tower with these passwords to be able to decrypt and use the vaulted files during playbook execution.
3. Click the green _+_ button.
4. Give your credential a name (for example _MyCred_), optionally a description and select your organization _ACME Corporation_.
5. Select _Machine_ as the credential type. This will present you with some additional fields to fill in. As you can see there are lots of different types of credentials which show you the power of Ansible Tower in its ability to handle all these types.
6. For username and password use the _EXACT_ same username and password you used in the previous exercise. You still remember them, right...
7. If you plan to deploy Linux based building blocks in your workflow then specify _sudo_ for _Privilege Escalation Method_, _root_ for _Privilege Escalation Username_ and the same password you used earlier for _Privilege Escalation Password_.
8. Click _SAVE_

> **_Notes:_**
>
>* Try to look up the password you just entered. As you will notice you can’t anymore. You can only replace it. Nice and safely stored :-)
>* If you want to check whether you typed the password correctly, you can do that only before you click SAVE.
>* We use the same _Machine_ credential for both Linux and Windows machines. This works, because on Windows, the account has admin rights and hence do not nmeed priviledge escalation.

[(top)](#table-of-contents)

## Create an Inventory

As you know by now you need an inventory of machines you want your playbooks to run against. In this part we are going to create a dynamic inventory. This is an inventory that will populate itself by syncing from an external source, in this case Azure. This means that as you create VM’s in Azure the inventory will be populated with the VM’s that you have created once you sync it. You need credentials for Azure to be able to do this, but these have been shared with you by the Ansible Tower Admin to use, remember?

1. Log into Ansible Tower as _linuxadmin_ (if you haven’t already)
2. Go to _Inventories_ in the left main menu, click the green _+_ button and choose _Inventory_.
3. Give your inventory a _NAME_ (ie _MyInventory_), optionally a _DESCRIPTION_, _ACME Corporation_ for _ORGANIZATION_ and click _SAVE_
4. Choose _SOURCES_ in the inventory menu and click the green _+_ button.
5. Give the source a _NAME_(suggestion: _Azure_) and optionally a _DESCRIPTION_. Choose _Microsoft Azure Resource Manager_ as the _SOURCE_ (note that the _CREDENTIAL_ is auto populated to the _Azure CREDENTIAL_, because that is the only one with that type you have access to).
6. Select _Overwrite_ in the _UPDATE OPTIONS_ (click the _?_ next to it to get an explanation on this option)
7. **Important**: In _SOURCE VARIABLES_, just below the “---”, so on line 2, type in the following:
    ```
    resource_groups: ansible_workshop_{guid}
    ```    
    This little trick will filter the hosts in Azure to those that are in your Azure resource group.
8. Click _SAVE_
9. Click the _SYNC ALL_ button. It will now start syncing your current VM’s in Azure into your inventory.
10. When the synchronization is done (the cloud icon next to the source stops blinking and is green), click the _HOSTS_ submenu item within the inventory. If all is well you see the VM you deployed in the previous step. If you see none or more than one, **that is not good! Stop right here! Talk to the instructor!**
11. When you select the host you see the DETAILS pane where you see vars that are generated from the Azure SOURCE and that you can use in your playbooks.
12. In the _GROUPS_ tab you also see that the host is automatically added to various groups that represent different types of Azure metadata. The resource_group is one, The OS is another, Also, the role you specified during the creation you now see come back here as well. We will use these groups to run the different playbooks against.

[(top)](#table-of-contents)

## Create Job Templates

You now add job templates for each node in the above workflow:
1. Log into Ansible Tower as _linuxadmin_ (if you haven’t already)
2. Go to _Templates_ in the left main menu, click the green + button and choose _Job Template_
3. Fill out the details like this:
    * _NAME_: choose the name of the playbook and make it the same as the node in the above workflow, for example _windows_create_user_
   * _DESCRIPTION:_ is optional and you can choose anything
   * _INVENTORY:_ The Inventory you just created
   * _PROJECT:_ Choose Windows or Linux, depending on the workflow node you are working on.
   * _PLAYBOOK:_ Choose the correct playbook from the list.
   * _CREDENTIAL:_ Choose the Machine Credential you just created.
   * _ONLY:_ for the job template _windows_create_user_, enable the option _Enable Concurrent Jobs_. What this does is enabling the possibility to have multiple jobs running from this template concurrently. As you can see from the workflow diagram, we want this. The shared job template _azure_create_vm_ has the same capability.
   * Click _SAVE_.

Repeat this exercise for each building block in your workflow. If might seem like a lot of work, but once you’ve done one or two, it becomes second nature and you can make them quite fast.

>_**Note:**_
>
>If multiple nodes in the above workflow execute the same job template, as for example with windows_create_user, you only need to denine the job template once. 

[(top)](#table-of-contents)

## Build Surveys

The used playbooks need a survey defined on the job template to be able to work. This way, the template will provide the playbook with user defined var’s. You create a survey by going to the job template details and click on _ADD SURVEY_. Here you can enter the specification of each field in the survey. When you’re done, click _SAVE_. Here is the specification for the survey of each job template that you need to make:

---
Survey for job template _**windows_create_dc**_

| PROMPT   | DESCRIPTION | ANSWER VARIABLE NAME | ANSWER TYPE | MIN - MAX LENGTH | REQUIRED |
| ---------|-------------|----------------------|:-----------:|:----------------:|:--------:|
| Name     | Domain Name | domain_name          | Text        | 4 - 30           | Yes      |
| Password | Password    | password             | Password    | 8 - 20           | Yes      |
	
---
Survey for job template _**windows_create_user**_

| PROMPT      | DESCRIPTION | ANSWER VARIABLE NAME | ANSWER TYPE | MIN - MAX LENGTH | REQUIRED |
| ------------|-------------|----------------------|:-----------:|:----------------:|:--------:|
| Domain Name | Domain Name | domain               | Text        | 4 - 30           | No       |
| Username    | Username    | username             | Text        | 4 - 30           | Yes      |
| Password    | Password    | password             | Password    | 8 - 20           | Yes      |
| Group       | Group Name  | group                | Multiple Choice (Single Select) | Domain Admins Users | Yes |

---
Survey for job template _**linux_join_domain**_

| PROMPT      | DESCRIPTION    | ANSWER VARIABLE NAME | ANSWER TYPE | MIN - MAX LENGTH | REQUIRED |
| ------------|----------------|----------------------|:-----------:|:----------------:|:--------:|
| Domain      | Domain Name    | domain               | Text        | 4 - 30           | No       |
| Name        | Admin username | ad_user              | Text        | 4 - 30           | No       |
| Password    | Admin Password | ad_password          | Password    | 8 - 20           | No       |	

> _**Note:**_
>
> Did you notice some fields in the survey specification are optional (REQ: No)? So what would happen if you keep them empty in the workflow? Here, a nice feature is demonstrated: The ability to pass data from one node in a workflow to nodes that follow it. How that works? Go look at the underlying playbooks in gitlab and search for the method “set_stats”. That should explain it. You can test it by actually leave all fields that are optional empty, which we will do below.

[(top)](#table-of-contents)

## Create workflow template

Now we finally have all the needed components to build a workflow. Phew! A workflow is a set of job templates (and thus playbooks) that are linked together in a certain specific way such that multiple job templates are executed in a predictable order. Each job template in a workflow is called a _node_.
1. Log into Ansible Tower as _linuxadmin_ (if you haven’t already).
2. Go to _Templates_ in the left main menu and click the green + button and choose _Workflow Template_
3. Fill out the details like this:
    * _NAME:_ name the workflow “create_windows_domain_with_linux_client” (just to show you can create really long template names ;-)
    * _DESCRIPTION:_ is optional and you can choose anything
    * _ORGANIZATION:_ Choose ACME Division 
    * _INVENTORY:_ Choose the Azure inventory you made previously
 4. Click _SAVE_.
 
 You are now brought to the _Workflow Visualizer_. Here you can graphically build out your workflow.
 5. Click on the _START_ node. A dialog will open where you can choose one of your job templates to add to the workflow.
 6. Choose _azure_create_vm_ by clicking on the name (it is greyed-out , but you can still select it by clicking the text. A known bug), scroll down and click _PROMPT_.
 7. You can now fill out the survey needed for this node. For the first _azure_create_vm_ node select the same values you used to create the test VM in a previous step. It will then reuse that VM.
 8. Click _SELECT_.
 9. You can now click the _START_ step again to add a step that runs parallel to the previously created step.
10. When you hover over one of the just created _azure_create_vm_ nodes 3 small icons appear:
    * A green icon you can click to add a node after this node
    * A red icon to remove this node
    * A blue icon to create a link from this node to another existing node. You create the link by clicking on the target node. You select if this path should be followed “Always”, “On Success” or “On Failure” and you click SAVE to establish the link.

11. You need to create 2 VM’s: One Windows VM with role “dcserver” and one VM with role “webserver” (the one you already made).

12. After you have added the nodes that create the needed VM’s in Azure for your workflow you want an inventory sync to happen because otherwise the playbooks that must operate on those VM’s have an empty inventory. To do this, add a node after the _azure_create_vm_ node(s) and, in the top of the dialog that appears, choose _INVENTORY SYNC_. Then you can choose what inventory source to sync and in what condition (Success, Failure, Always). See the example workflow above to see how that looks like. You first add (green icon) an “inventory sync” node after the first “azure_create_vm” node and then you link (blue icon) the other “azure_create_vm” node to it. Easy!
13. Make sure you use a 2-part domain name for the node _windows_create_domain_. So, _example.com_ and not _example_.
14. The node _windows_create_user_ that precedes the node _linux_join_domain_ must create a user that is a member of the _domain_admins_ group.
15. Make sure you choose usernames that are unique and you have not used before.
16. When done with building the workflow, click SAVE.
17. Do NOT fill out the survey for the linux_join_domain node. Why not? Did you notice some fields in the survey specification are optional (REQ: NO)? So what would happen if you keep them empty? Here, a nice feature is demonstrated: The ability to pass data from one node to nodes that follow it. How that works? Go look at the underlying playbooks in gitlab and search for the method “set_stats”. That should explain it. You can test it by actually leave all field that are optional empty, which we do here.

With this simple (...) set of instructions, you should be able to build out the workflow for this exercise, so have a go at it!

>_**Notes:**_
>
>* You can move the workflow diagram on you screen by click-and-drag on an empty spot in the workflow area.
>* You can zoom in or out by doing a two-finger drag (without click!) in de workflow area up or down.
>* You can use both these actions also by clicking on the gear icon in the top right of the workflow area.

[(top)](#table-of-contents)

## Finalize workflow template

Now, add a notification to the workflow. Notifications are a way to get notified of the result of a job run: Success or Failure. There are many notification methods and one of them is _Slack_. We have already set up a notification to a Slack channel for you to use. You can set various notifications on various levels (project, org, job template, etc) so it’s a quite powerful feature. Let’s add a Slack notification to the workflow you made:
1. Log in as “admin” (only certain roles, such as admin and auditor, can manage notifications)
2. Go to the workflow template details that you have created and click on NOTIFICATIONS. You see the list of notifications is currently one: the defined slack channel.
3. Enable the notification for both SUCCESS and FAILURE.

> _**Note:**_
>
> The slack channel is displayed on the presentation screen. If it’s not, notify the instructor: he forgot ;-).

Last (but not least), assign the “Execute” role to the user named  “user” in PERMISSIONS. As you are already experienced on assigning roles, we will not elaborate on the details :-).

[(top)](#table-of-contents)

## Execute workflow template

Now we finally are ready to give the workflow a run for its money!
You can run this workflow either as user _linuxadmin_ or as user _user_. The difference is that _linuxadmin_ has access to the logging, the inventory, etc, so if anything goes wrong you can immediately see what it is. When you execute the workflow as _user_, you only see the workflow and the ability to execute it. You can follow the execution, see if it succeeds or fails, but you have no access to the logging.
1. Log in as either _linuxadmin_ or _user_. (You now know the difference).
2. Find the workflow in the list of templates and press the rocket icon next to it.

The screen will move to a graphical representation of the workflow and you see continued feedback where the execution is and what the result of each node’s execution is: a green outline means SUCCESS. A red outline FAILURE. You can click on the “DETAILS” and it will bring you to the logging if you are logged in as _linuxadmin_.

Hopefully your workflow will finish in one go. If it does, in the logging of the last step (the haproxy node) you find the details to access the website (basically, the public IP of the load balancer VM).

> _**Note:**_
>
> If something has gone wrong, probably a node is displayed in red. See what’s wrong, fix it, and restart the workflow job, just like you can restart a template job. You do this by going into the workflow jobs details page and press the run icon. This will restart the workflow with the previously entered survey. You don’t need to worry about creating VMs multiple times. Ansible will take care of just creating the VMs that do not exist yet.

[(top)](#table-of-contents)

## Cleanup

After you have recovered from the amazement of your result it is time to clean up. We made a special job template for that:
1. Log in as “linuxadmin”
2. Find the job template called “azure_delete_all” and execute it. It will ask for a confirmation (duh!) and if you confirm, it will remove all the VM’s you have created in Azure.

> _**Note:**_
>
> If you plan to also do the other exercise, it is mandatory that you do this.

[(top)](#table-of-contents)

## Thank you!

We hope you got a good sense of the basics around Ansible Tower. We do invite you to continue learning Ansible and Ansible Tower. Here are some great resources you can use to further develop you Ansible skills:

Quick Start Video:
https://www.ansible.com/quick-start-video

Ansible Documentation:
https://docs.ansible.com/

Ansible Galaxy:
https://galaxy.ansible.com/

List of all modules:
https://docs.ansible.com/ansible/latest/modules/modules_by_category.html

Get a Ansible Tower Trial license:
https://www.ansible.com/products/tower/trial

Manage Windows like Linux with Ansible:
https://www.youtube.com/watch?v=FEdXUv02Dbg

All playbooks are permanently available on:
https://github.com/fvzwieten

## Happy automating! (with Ansible Tower!)
