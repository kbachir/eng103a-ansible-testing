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

## Installation and setup
Follow these steps to get the following services we used. Disclaimer: some of these commands might be updated, so double check with the official documentation. The operating system was Ubuntu on the instances (version 18.04). The virtual machines were all also hosted on AWS and made in a T2.medium.
### Jenkins
To set up Jenkins go into a Linux instance and run the following commands:
- `sudo apt update && sudo apt upgrade -y` Updates and upgrades the instance. Also used to check whether there is internet access in the instance
- `sudo apt install default-jre -y` Installs latest version of Java (in our case it was 11.0.14.1). Jenkins uses Java so it's essential. You can also use `java -version` to check the version. It should look like the image below.
![diagram](https://cdn.discordapp.com/attachments/958316995156267068/964178320382099506/unknown.png)
- `wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -` This imports the GPG keys of the Jenkins repository.
- `sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'` This adds the Jenkins repository to the system.
- `sudo apt install jenkins` To install the latest version of Jenkins.
- `systemctl status jenkins` To display the status of Jenkins. It should display the following, with different dates, time, etc.
![diagram](https://cdn.discordapp.com/attachments/958316995156267068/964180361984741416/unknown.png)
- The firewall on your instance needs to be adjusted in order to allow Jenkins to work. In the case of AWS you need to change the security groups to allow for port `8080` access.
- In your browser, type your domain IP followed by port `8080`, in the format `http://your_ip_or_domain:8080`. This should take you to an unlock Jenkins page which looks like this:
![diagram](https://linuxize.com/post/how-to-install-jenkins-on-ubuntu-18-04/unlock-jenkins_huff7c186b4c9370e26f4ba03e0bde0db7_51753_768x0_resize_q75_lanczos.jpg)
- Type `sudo cat /var/lib/jenkins/secrets/initialAdminPassword` into the terminal and put the output into the password box.
- Time to install some plugins.Plugins needed are all the default plugins, ansible, parameterised trigger plugin, and dashboard view. Once you click install, it will take some time to install all the needed plugins. Once it's done click `save and continue`.
- The page will prompt you to create a Admin user, once this is done continue, copy the Jenkins URL and click `save and finish`.
- Jenkins is now ready, click on `start using jenkins` to go to the jenkins dashboard.

### Grafana
To set up Grafana on the linux instance, follow the steps below.
- If it's a new instance make sure you use `sudo apt update && sudo apt upgrade -y`.
- `wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -` To download the Grafana GPG key and pipe the output.
- `sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"` To add the Grafana repository to your APT sources.
- `apt-cache policy grafana` This will show the version of grafana you are going to install. It should look like:
![diagram](https://cdn.discordapp.com/attachments/814569440511262800/964192491828047942/unknown.png)
- `sudo apt install grafana` This will install Grafana
- `sudo systemctl start grafana-server` to start the Grafana server
- `sudo systemctl status grafana-server` Display the status of the grafana server
- `sudo systemctl enable grafana-server` This enables the service to automatically start Grafana on boot
- Navigate to `https://your_domain_IP:3000`, your domain IP will be the IP of the instance or your localhost depending on how you are setting up Grafana.This should display a page with the Grafana name and logo, an email/username, and password form. It should look like the image below.
![diagram](https://assets.digitalocean.com/articles/66778/Grafana_Login.png)
- Enter `admin` into the username and password fields and click login. This will lead to a page where you will need to make a more secure password. Once this is done you will be brought to the grafana home dashboard.
- In order to monitor the instance you need to monitor Prometheus. So slick on the `cogwheel` on the grafana dashboard. Click on `data sources`. Click on `add data source`. Select prometheus as the type. Set the appropriate Prometheus server URL, if you have it running on the same instance do `http://localhost:9090` otherwise put in the instance IP with the correct port.Access method   . Click save and test to save the new data source.
### Prometheus
- `sudo apt update && sudo apt upgrade -y` 
- `curl -LO url -LO https://github.com/prometheus/prometheus/releases/download/v2.22.0/prometheus-2.22.0.linux-amd64.tar.gz` To download the Prometheus GPG key
- `tar -xvf prometheus-2.22.0.linux-amd64.tar.gz` To extract from the download and create a new directory
- `mv prometheus-2.22.0.linux-amd64 prometheus-files` Rename the extracted folder to prometheus-files
- Create a prometheus user, required directories and make Prometheus the user as the owner of those directories
``sudo useradd --no-create-home --shell /bin/false prometheus``

``sudo mkdir /etc/prometheus``

``sudo mkdir /var/lib/prometheus``

``sudo chown prometheus:prometheus /etc/prometheus``

``sudo chown prometheus:prometheus /var/lib/prometheus``

- Copy prometheus and promtool binary from prometheus-files folder to /usr/local/bin and change the ownership to prometheus user.

``sudo cp prometheus-files/prometheus /usr/local/bin/``

``sudo cp prometheus-files/promtool /usr/local/bin/``

``sudo chown prometheus:prometheus /usr/local/bin/prometheus``

``sudo chown prometheus:prometheus /usr/local/bin/promtool``

- Move the consoles and console_libraries directories from prometheus-files to /etc/prometheus folder and change the ownership to prometheus user.

``sudo cp -r prometheus-files/consoles /etc/prometheus``

``sudo cp -r prometheus-files/console_libraries /etc/prometheus``

``sudo chown -R prometheus:prometheus /etc/prometheus/consoles``

``sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries``

All the prometheus configurations should be present in /etc/prometheus/prometheus.yml file.

- `sudo nano /etc/prometheus/prometheus.yml`  To create the yml file, and put the content below into it. The agent ip should be in the agent target. Save it when you're done.
```
global:
  scrape_interval: 10s

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']
  - job_name: "agent"
    static_configs:
      - targets: ["agent_IP:9100"]
```
`sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml` To change the ownership of the file to prometheus user.

- `sudo nano /etc/systemd/system/prometheus.service` To create a prometheus service file. Put the content below into the file.

```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```
- `sudo systemctl daemon-reload` and `sudo systemctl start prometheus` To reload the systemd service to register the prometheus service and start the prometheus service.
- `sudo systemctl status prometheus` to check the status of prometheus.
- Now you should be able to access the prometheus UI by going to `http://<prometheus-ip>:9090/graph`.

On prometheus make sure the connection with the agent is up on targets.