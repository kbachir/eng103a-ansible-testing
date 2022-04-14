# eng103a ansible sever patching Readme
This readme is meant to be used alongside another readme which talks more in detail what the playbooks do.

The aim of this project was to run tests on services using ansible playbooks, if there were issues found with the services the pipeline would automatically trigger a playbook which would fix the issues. The status of the services being run can be monitored using Grafana and pulling data from the playbook. The diagram below shows the project layout. Jenkins is used to automate the patching. 
![diagram](https://cdn.discordapp.com/attachments/958316995156267068/963098096433262612/unknown.png)
The diagram below shows the flow of the Jenkins jobs.Run the job which uses Adhoc command to stop nginx (job 1), this is used to show on grafana that it has stopped and so you can run test properly.You run the `test.yml` playbook (job 2), this will run some tests. If it fails the test then Jenkins will run the `test_solution.yml` playbook (job 2.1) , this will fix the instance.The playbook `svc.yml`  (job 3) will be run after job 2.1 or job 2. This will return written output of the services status, which will be pulled by Grafana and used in logs.
![diagram](https://cdn.discordapp.com/attachments/958316995156267068/964165977958924288/Jenkins_Jobs.png)

## What is Grafana?
Grafana is a visualisation tool, for supported data sources (in our case Prometheus). We are using grafana to create dashboards to track the four golden rules of monitoring, these are:
- latency
- traffic 
- errors 
- saturation

## What is Prometheus?
Prometheus is an open source monitoring system, that can pull server logs and query them so we can monitor various parameters in real time. We are using node exporter to feed the data into Prometheus, which is visualised by Grafana. 
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

## Setup 
Follow these steps to get the following services we used up and running. Disclaimer: some of these commands might be updated, so double check with the official documentation. The operating system was Ubuntu on the instances (version 18.04). The virtual machines were all also hosted on AWS.
### Jenkins
To set up Jenkins go into a Linux instance and run the following commands:
- `sudo apt update && sudo apt upgrade -y` Updates and upgrades the instance. Also used to check whether there is internet access in the instance
- `sudo apt install default-jre -y` Installs latest version of Java (in our case it was 11.0.14.1). Jenkins uses Java so it's essential. You can also use `java -version` to check the version. It should look like the image below.
![diagram](https://cdn.discordapp.com/attachments/958316995156267068/964178320382099506/unknown.png)
- `wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -` This imports the GPG keys of the Jenkins repository.
- `sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'` This adds the Jenkins repository to the system.
- `sudo apt install jenkins` To install the latest version of Jenkins.
- `systemctl status jenkins` To display the status of Jenkins. It should display

Plugins needed are all the default plugins, ansible, parameterised trigger plugin, and dashboard view.
### Grafana

### Prometheus
