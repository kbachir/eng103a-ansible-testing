# eg103a ansible sever patching Readme
The aim of this project was to run tests on services using ansible playbooks, if there were issues found with the services the pipeline would automatically trigger a playbook which would fix the issues. The status of the services being run can be monitored using Grafana and pulling data from the playbook. The diagram below shows the project layout.
![diagram](https://cdn.discordapp.com/attachments/958316995156267068/963098096433262612/unknown.png)
Jenkins is used to automate the patching.

## What is server patching

## Talk about playbooks
There are three Yaml playbooks that we need to discuss. 

- Firstly, `test.yml` runs the initial tests on the agent instances. It checks the timezone and confirms that it is UTC. It checks whether nginx is installed, and finally it confirms that it is listening on port 80.

- `test_solution.yml` is used to correct any issues on the agent instances, it will set the timezone to UTC, install and start nginx.

- `test_svc.yml` is used to gather written information about the instances. It will check whether nginx is running, then depending on that it will send a message saying that its' running as expected or nginx is down.


## 