# eng103a ansible sever patching Readme
The aim of this project was to run tests on services using ansible playbooks, if there were issues found with the services the pipeline would automatically trigger a playbook which would fix the issues. The status of the services being run can be monitored using Grafana and pulling data from the playbook. The diagram below shows the project layout.
![diagram](https://cdn.discordapp.com/attachments/958316995156267068/963098096433262612/unknown.png)
Jenkins is used to automate the patching.

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
- Simple(relatively) to set up and use
- Agentless 
- Efficient
- Reliable

