# eng103a ansible sever patching Readme
This readme is meant to be used alongside another readme which talks more in detail what the playbooks do.

The aim of this project was to run tests on services using ansible playbooks, if there were issues found with the services the pipeline would automatically trigger a playbook which would fix the issues. The status of the services being run can be monitored using Grafana and pulling data from the playbook. The diagram below shows the project layout. Jenkins is used to automate the patching. 
![diagram](https://cdn.discordapp.com/attachments/958316995156267068/963098096433262612/unknown.png)
The diagram below shows the flow of the Jenkins jobs. You start with pulling the code from the the dev branch, then you run the `test.yml` playbook, this will run some tests. If the instance passes the tests, then it merges with main. If it fails the test then Jenkins will run the `test_solution.yml` playbook, this will fix the instance, then it will be merged with main. After it is merged with main the playbook `svc.yml` will be run. This will return written output of the services status, which will be pulled by Grafana and used in logs.
![diagram](https://media.discordapp.net/attachments/938459479270359051/963759752381345822/Jenkins_Jobs.png?width=960&height=540)
## What is server patching?
Patching is a process to repair a vulnerability, a flaw that is identified after the release of an application, fix bugs and add new features. 

## What is server drift?
Server drift (also known as configuration drift) happens when make modifications on a system over time the configuration of the system changes. Theses changes could be very minor but over time the system will change. 

An example would be beneficial. Imagine you create two instances( a and b), both with the same OS, and the same apps. Over time if you only update one of them (a), it will look different if you compare it to (b).

## Why are we using ansible for these jobs?
Ansible is an infrastructure as code open source provisioning and, configuration management tool. In this case we are using it for configuration management tool. We use ansible because it allows us to make changes to a large amount of instances with relatively little effort. Before ansible if you wanted to update some software on several instances you would need to go into the instances individually and then update them. This would take a lot of time and could be unreliable if done by a human.

Using ansible, as long as we have a file which gives us information on how to connect to the instances and write some playbooks in Yaml format (instructions on what commands to run), we can perform actions on thousands of instances if needed.

## The playbooks being used

Some of the benefits of ansible include:
- Simple(relatively) to set up and use. Anisble's playbooks require no special coding skill to use.
- Agentless. No need to install any other software or firewall ports on the client systems you want to automate. You also don't have to set up a separate management structure.
- Efficient. No extra software needed so there's more room for application resources on your server.
- Reliable. Executing a playbook can be done one thousand times, and it will always be done the same way. 

