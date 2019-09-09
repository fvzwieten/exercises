# Exercise 1: Mixed Web Stack
## Table of Contents
- [Introduction](#introduction)
- [Goal](#goal)
- [Permissions](#permissions)
- [Test Deploy a VM](#test-deploy-a-vm)
- [Create Machine Credential](#Create-Machine-Credential)
- [Create an Inventory](#Create-an-Inventory)
- [Create Job Templates](#Create-Job-Templates)
- [Finalize workflow template](#Finalize-workflow-template)
- [Execute the workflow](#Execute-the-workflow)
- [Cleanup](#Cleanup)

## Introduction
Welcome to exercise 1 of the Ansible for Linux & Windows Hands-on Lab!

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


The url of the **git server** where your repositories live is: https://control-{guid}.rhpds.opentlc.com/gitea
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
![example workflow](https://github.com/fvzwieten/media/blob/master/exercise1.png "Example Workflow")
Remember, this is just an example!

* If you choose to use Linux for your web server, you must use the playbooks linux_deploy_apache and linux_deploy_website as your building blocks of choice
* If you choose to use Windows for your web server, you must use the playbooks windows_deploy_iis and windows_deploy_website.
* You can deploy multiple web servers. Even, like the example shows, one (or more) Windows based web servers and one (or more) Linux based web servers. They will all serve the same example website.
* The load balancer will do round-robin load balancing across all web servers, independent of the underlying technology stack. So, sometimes you are served using a Linux web server and sometimes using a Windows web server. And you won’t be able to tell the difference in the deployed app. How neat is that!
* Same thing for the database: You either choose the playbooks windows_deploy_mariadb and windows_deploy_db for a Windows based database server or linux_deploy_mariadb and linux_deploy_db for a Linux based one.
* You actually deploy a real working website where you can maintain users that are stored in the database.

> **_Notes:_**
>
>1. If you deploy multiple database servers, which does not make sense at all, the web servers will only use the first one in the inventory.
>2. The load balancer is always Linux based. This is because the Windows Network Load Balancer (WNLB) is not supported on Azure and haproxy is not available for Windows. Using the Azure load balancer would have been a solution, but defeats the scope of this lab.
>3. You are free to deploy a stack with just one web server and then a load balancer is not needed and you can thus ignore the instructions on the load balancer part, but that would be less fun!

[(top)](#table-of-contents)

## Permissions

_webadmin_ currently only has access to the Azure job templates as shared by the Ansible Tower _admin_. So, you need to grant access to the Windows and/or Linux projects, depending on how you want to build your stack.

If you need Linux playbooks to build your stack:
1. Log into Ansible Tower with the _linuxadmin_ account
2. Go to _Projects_ in the left main menu. You see the projects this account has access to.
3. Select _Linux_. You see all the details of this project, including the url of the playbook repository on your git server.
4. In the _projects_ menu, select _PERMISSIONS_ and click the green _+_ button.
5. Select the _webadmin_ user
6. In the lower menu, choose _Use_ as assigned role (click in the input field for a list)
7. In the lower menu, choose _Update_ as assigned role (click in the input field for a list).
8. Click _Save_
9. Log out of Ansible Tower. That is the _switch_ icon in the upper right corner of the Tower UI, next to the _book_ icon.

If you (also) need Windows playbooks to build your stack, repeat the previous step where you replace _linux_ with _windows_.

> **_Note:_**
>
>If you want to assign roles to teams instead of users, like you did just now (e.g. _Web Team_), you need to be a Ansible Tower _Global Admin_.

[(top)](#table-of-contents)

## Test Deploy a VM

Nothing stops us from already deploying a VM in Azure. You can use that VM in the later workflow. So, let’s execute a playbook that will do just that and see how that works:
1. Log into Ansible Tower as _webadmin_
2. Check in _Projects_ if you now see the Linux/Windows projects based on the permissions you assigned earlier.
3. Go to _Templates_ in the left main menu. Here, you see a list of job templates that begin with azure_ and are shared with you and you are allowed to execute.
4. Next to each template you see a _run_ icon in the form of a flying rocket. Click on the one next to the job template called _azure_create_vm_.
5. You will be presented with a _survey_. A survey is a simple form to ask for input parameters for this run. Each field is self explanatory. The password field is already prefilled with the password you also use to log into tower and git. You are free to change it, but then please remember it for later.
6. Fill all fields and press _NEXT_. Choose values that you can later use in your workflow, so use a role you need. Remember all input you use here. You need them later! Also make sure you enter valid values. Now click _NEXT_.
7. You get to (p)review what you entered. You also see the var names that will be populated with these values in the playbooks. Press _LAUNCH_ when all is good.
8. You now see the _JOBS_ window appear with logging in the right pane as the job executes. The left pane shows all metadata of the job.

> **_Notes:_**
>
>* You can safely ignore the purple warning message at the beginning of the logging.
>* The job will take some time to execute. A Windows VM deployment will typically take longer than a Linux VM (on Azure). You can continue with the next exercise or take a look at the playbooks in your git server while you wait and go back to this job at any time by going to the _My View_ main menu item and look up this job’s details. As you can see each Job has been given a unique number. Select _My Jobs_ to only see your own jobs.
>* When the job completes it will report in the logging the public IP address of the VM. You can log into this VM with either RDP or SSH (depending on the OS you’ve chosen) and the username/password you entered in the survey.
>* If, for some reason you want to remove this VM, you can execute the job template called azure_delete_vm. It will ask for the VM Name and will delete that VM from Azure.
>* Have a look at the playbooks in the ansible4azure repository in your git server. They showcase how you can use Ansible with Azure.
>* If the browser window is too small, the interface will automatically switch to mobile view. Left and Right become Up and Down.
>* You have write access to the git repositories. Feel free to customize the playbooks to your own liking. Remember though that this server will be deleted after the workshop..

[(top)](#table-of-contents)

## Create Machine Credential

When you deploy machines, you needed to specify credentials to be able to log in to it. You also need credentials to be able to run Ansible playbooks _in_ the VM’s OS. Although the Windows admin and/or Linux admin could share their credentials to do that, that would mean you can log into their other machines as well. Better to create your own machine credential then! Here we go:
1. Log into Ansible Tower as _webadmin_ (if you haven’t already)
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

1. Log into Ansible Tower as _webadmin_ (if you haven’t already)
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

Depending on how you want to deploy _your_ version of the stack, you now add job templates for each node in your workflow.
1. Log into Ansible Tower as _webadmin_ (if you haven’t already)
2. Go to _Templates_ in the left main menu and click the green _+_ button and choose _Job Template_
3. Fill out like this:
    - _NAME_: choose the name of the playbook and make it the same as the node in the above workflow, for example “linux_deploy_apache”
    - _DESCRIPTION_: is optional and you can choose anything
    - _INVENTORY_: The Inventory you just created
    - _PROJECT_: Choose _Windows_ or _Linux_, depending on the building block.
    - _PLAYBOOK_: Choose the correct playbook from the dropdown list.
    - _CREDENTIAL_: Choose the Machine Credential you just created.

      **Important!** Certain playbooks use vaulted files that contain resource credentials, as explained above. Those vaults are encrypted using ansible-vault. To be able to decrypt them a vault credential needs to be specified when used in a playbook. The *_website and the *_db playbooks need the “Database Access” vault credential and the linux_deploy_haproxy playbook needs the “Load Balancers” vault credential. You can specify multiple credentials for a playbook (makes sense, right?). To do this open the searchbox of the Credential field again and choose “Vault” in the dropdown. Now you see the Vault credentials you have access to and you can choose the one you need for the playbook at hand.
4. Click SAVE.

Repeat these steps for each building block in your workflow. It might seem like a lot of work, but once you’ve done one or two, it becomes second nature and you can make them quite fast :-)

> **_Note:_**
>
>If multiple nodes in your workflow execute the same job template, as is for example with windows_deploy_website and/or linux_deploy_website, you only need to define the job template once. 

[(top)](#table-of-contents)

## Create workflow template

Now we finally have all the needed components to build a workflow. Phew! A _workflow_ is a set of _job templates_ (and thus _playbooks_) that are linked together in a certain specific way such that multiple job templates are executed in a predictable order. Each job template in a workflow is called a _node_. Nodes are connected using _paths_. A _path_ can be followed if the job template finshed either succesfully and/or not succesfully.

1. Log into Ansible Tower as _webadmin_ (if you haven’t already).
2. Go to _Templates_ in the left main menu and click the green + button and choose _Workflow Template_
3. Fill out like this:
    - NAME: name the workflow _Deploy Web Stack_
    - DESCRIPTION: is optional and you can choose anything
    - ORGANIZATION: Choose _ACME Corporation_ 
    - INVENTORY: Choose the _Azure_ inventory you made previously
4. Click _SAVE_

You are now brought to the Workflow Visualizer. Here you can graphically build out your workflow.
1. Click on the START node. A dialog will open where you can choose one of your job templates to add to the workflow.
2. Choose “azure_create_vm” by clicking on the text (it is greyed-out , but you can still select it. A known bug), scroll down and click PROMPT. You can now fill in all the fields needed for this node.
3. For the first azure_create_vm node select the same field values you used to create the test VM in the previous step. It will then reuse that VM.
4. Click SELECT. A node has been added to the workflow.
5. You can now click the START node again to add a step that runs parallel to the previous created step. This way you can, for example in this workflow, create multiple VM’s in parallel.
6. Now first create all Azure VM’s you need and specify the correct OS, role and credentials. (Suggested vm names: winweb{x}, linweb{x}, windb, lindb, linlb).
6. When you hover over the just created node 3 small icons appear:
    - A green icon you can click to add a node after this node
    - A red icon to remove this node
    - A blue icon to create a link from this node to another existing node. You create the link by clicking on the node to link to. You select if this path should be followed “Always”, “On Success” or “On Failure” and you click SAVE to establish the link.
    
**Important!** After you have added the nodes that create the needed VM’s in Azure for your stack you want an inventory sync to happen because otherwise the playbooks that must operate on those VM’s have an empty inventory. To do this, add a node after the _azure_create_vm_ node(s) and, in the top of the dialog that appears, choose _INVENTORY SYNC_. Then you can choose what inventory source to sync (Azure) and in what condition (Success, Failure, Always). See the example workflow above to see how that looks like. You first add an “inventory sync” node after the first “azure_create_vm” node and then you _link_ the other _azure_create_vm_ nodes to it. Easy!

When done with building your workflow, click SAVE.

> **_Notes:_**
>
>You can move the workflow diagram on your screen by click-and-drag on an empty spot in the workflow area.
>You can zoom in or out by doing a two-finger drag (without click!) in de workflow area up or down.
>You can also do both these actions by clicking on the gear icon in the top right of the workflow area.

[(top)](#table-of-contents)

## Finalize workflow template

We are almost there! Ony three things left to do, and one is to add a survey to the workflow template. Why would we need that? Well, the web app needs a name, of course! This is optional and if you don’t create a survey, the website playbooks are smart enough to define a default name for the app. If your curious how that's done, just have a look in the specific playbooks in the git repo.

1. Log into Ansible Tower as _webadmin_ (if you haven’t already).
2. Go to the workflow template details that you have created and click ADD SURVEY.
3. Create a field:
    - Prompt: Name
    - Description: Application Name
    - Answer variable name: _appname_
    - Answer type: _text_
    - Minimum length: 4, Maximum length: 40
    - Default answer: whatever you like
    - Required: _no_
    - Click _ADD_ and then _SAVE_

Next, add a notification to the workflow. Notifications are a way to get notified of the result of a job run: Success or Failure. There are many notification methods and one of them is slack. We have already set up a notification to a slack channel for you to use. You can set various notifications on various levels (project, org, job template, etc) so it’s quite a powerful feature. Let’s add a slack notification to the workflow you’ve made:

1. Log into Ansible Tower as “admin” (only certain roles, such as admin and auditor, can manage notifications)
2. Go to the workflow template details that you have created and click on _NOTIFICATIONS_. You see the list of notifications is currently one: the defined slack channel.
3. Enable this notification for both SUCCESS and FAILURE by sliding the sliders to "On".

    > **_Notes:_**
    >
    >The slack channel should be displayed on the presentation screen. If it’s not, notify the instructor: he forgot..

Last (but not least), assign the _Execute_ role to the user named _user_ in _PERMISSIONS_. As you are already experienced on assigning roles, we will not elaborate on the details :-).

[(top)](#table-of-contents)


## Execute the workflow

Now we finally are ready to give the workflow a run for it’s money!

You can run this workflow either as user _webadmin_ or as user _user_. The difference is that _webadmin_ has access to the logging, the inventory, etc, so if anything goes wrong you can immediately see what it is. When you execute the workflow as _user_, you only see the workflow and the ability to execute it. You can follow the execution, see if it succeeds or fails, but you have no access to the logging.

1. Log into Ansible Tower as either _webadmin_ or _user_. (you now know the difference)
2. Find the workflow in the list of templates and press the rocket icon next to it. If all is well it should ask for the application name because you just created that survey. Enter (optionally) a nice description, click NEXT and then, finally, click _LAUNCH_ !!!

* The screen will now move to a graphical representation of the workflow and you see continued feedback where the execution is and what the result of each node’s execution is: a green outline means SUCCESS. A red outline FAILURE. The black dot means the node is currently executing. You can click on the _DETAILS_ and it will bring you to the logging (if you are logged in as _webadmin_).
* Notice the azure_create_vm jobs run concurrently. That is a job template property.
* Hopefully your workflow will finish in one go. If it does, in the logging of the last step (the haproxy node) you find the details to access the website (basically, the public IP of the load balancer VM).
* It will take some time to complete. Studying the playbooks while waiting is a good idea..
* Finished? Well Done! Have some fun with the website!
* You can also access the load-balancer statics page (see logging for the details). It will ask you for a password. That password you already know.

> **_Note:_**
>
> If something has gone wrong, probably a node is displayed outlined in red. See what’s wrong in the logging, fix it, and restart the workflow job, just like you can restart a template job. You do this by going into the workflow jobs details page and press the run icon (so, NOT the job template details!). This will restart the workflow with the previously entered survey. You don’t need to worry about creating VMs multiple times. Ansible will take care of just creating the VMs that do not exist yet. It is omnipotent in that regard.

## Cleanup

After you have recovered from the amazement of your result it is time to clean up. We made a special job template for that:

1. Log into Ansiuble tower as _webadmin_ (if you haven't already)
2. Find the job template called _azure_delete_all_ and execute it. It will ask for a confirmation (duh!) and if you confirm, it 

will remove all the VM’s you have created in Azure.

> **_Important!:_**
>
> If you plan to do another exercise, it is mandatory that you do this step!

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
