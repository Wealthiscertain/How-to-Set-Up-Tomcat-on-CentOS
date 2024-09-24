# How-to-Set-Up-Tomcat-on-CentOS

![image](https://github.com/user-attachments/assets/f2ae33db-8a88-4c34-b35b-b37d83e84537)

In this guide, we'll learn how to set up Apache Tomcat, focusing on how to manage it with `systemctl`. While many services can be managed by `systemctl` by default, some-like Tomcat-don't come pre-configured. We'll walk through the process of building a `systemctl` service for Tomcat, a Java-based web server and servlet container.

### **Tools Used:**
- **Vagrant:** Automates the creation and management of virtual environments.
- **CentOS:** A free, enterprise-class Linux distribution that is great for server environments.
- **Git Bash:** A command-line tool that allows users to run Git commands and other Unix-based utilities on Windows.
- **VSCode:** A powerful source code editor for development.
- **Oracle VM VirtualBox Manager:** Virtualization software for running multiple virtual machines.

### **What is Tomcat?**
Tomcat is an open-source servlet container developed by the Apache Software Foundation. It's designed to host Java-based applications and differs from Apache HTTP Server (`httpd`), which is used to serve general web content. We'll cover installing and running Tomcat on a CentOS virtual machine (VM).

**Step-by-Step Setup**

### **Step 1: Set Up Your CentOS VM**
To begin, you'll need a CentOS VM. If you don't have one yet, refer to our previous guide on [setting up a CentOS VM](https://medium.com/@wealthiscertain/step-by-step-guide-to-creating-a-centos-vm-with-vagrant-9b67ccb5916e). Once your VM is running, log in and switch to the root user:

```
vagrant ssh
sudo -i
```

![image](https://github.com/user-attachments/assets/f59883ac-6019-494f-a2af-fb0d6c0abbf4)

### **Step 2: Install httpd on the CentOS VM**
First, install Apache HTTP Server (`httpd`), which can be managed by `systemctl`:

```
dnf install httpd -y
```

![image](https://github.com/user-attachments/assets/544b1d5a-ddf8-4cf3-bb86-82634b7d5b1a)

![image](https://github.com/user-attachments/assets/23ffa96a-e0aa-4228-a6fb-52731590a9d3)

- Check the status of the httpd service

```
systemctl status httpd
```

![image](https://github.com/user-attachments/assets/c7109bb7-788d-4f9e-8b5b-6f60d4df7704)

### **Step 3: Understanding the Systemd Service for httpd**
To see how `systemctl` manages services like `httpd`, navigate to the configuration file:

```
cat /usr/lib/systemd/system/httpd.service
```

![image](https://github.com/user-attachments/assets/be6866d5-fb63-4f84-a46a-35ebaafab1c7)

In this file, you'll find three important directives: `Unit`, `Service`, and `Install`.

### **Step 4: Download and Install Tomcat**
Next, let's install Apache Tomcat. Go to the [Apache Tomcat website](https://medium.com/r/?url=https%3A%2F%2Ftomcat.apache.org) and download the latest version of Tomcat (in this case, Tomcat 10).

![image](https://github.com/user-attachments/assets/ef55dd69-760a-4b8d-9819-594aa878f2ff)

![image](https://github.com/user-attachments/assets/0cc89d50-69da-4a7c-b607-c1d6165a00f3)

```
wget https://archive.apache.org/dist/tomcat/tomcat-10/v10.1.28/bin/apache-tomcat-10.1.28.tar.gz
```

![image](https://github.com/user-attachments/assets/8c55726c-4fef-4456-9e0a-b1f9371d44c3)

- Extract the downloaded archive:

```
tar xzvf apache-tomcat-10.1.28.tar.gz
```

It is in `tar` format, `tar` is a command in Linux used for creating, viewing, and extracting files from archives.

![image](https://github.com/user-attachments/assets/66d4accf-207d-4415-bd4c-8bb94d910d1a)

### **Step 5: Install Java Dependencies**
Tomcat requires Java to run, so let's install OpenJDK 17:

```
dnf install java-17-openjdk -y
```

![image](https://github.com/user-attachments/assets/4c966c7a-b283-44f9-9982-f6aacf45a9fd)

![image](https://github.com/user-attachments/assets/5381a69d-b763-4681-9587-8c28a2c57c47)

- Verify the Java installation:

```
java -version
```

![image](https://github.com/user-attachments/assets/6d1404fd-0f87-4b80-88cb-6e869e4e2844)

### **Step 6: Start the Tomcat Service**
To start Tomcat, navigate to its `bin` directory and run the `startup.sh` script:

```
bin/startup.sh
```

![image](https://github.com/user-attachments/assets/f9483acb-1bc5-4fce-a8ac-d37e84673b66)

To verify Tomcat is running, use the following command:

```
ps -ef | grep tomcat
```

Switch back to root user

```
cd ..
```

![image](https://github.com/user-attachments/assets/79f37bc5-2bed-400e-9556-1ab6e1ac7cb6)

### **Step 7: Set Up Tomcat as a Non-Root User**
Tomcat should run as a non-root user. Let's create a dedicated `tomcat` user and move the Tomcat installation to a new directory:

1. Create a new `tomcat` user:

```
useradd -r -m -d /opt/tomcat -s /bin/false tomcat
```

2. **Copy the Tomcat files to the `/opt/tomcat` directory:**

```
cp -r apache-tomcat-10.1.28/* /opt/tomcat/
```

3. Assign ownership of the directory to the `tomcat` user:

```
chown -R tomcat:tomcat /opt/tomcat/
```

![image](https://github.com/user-attachments/assets/a2c236ed-da6c-4e09-8e7e-c2c3312ad88e)

### **Step 8: Create a Systemd Service for Tomcat**

Now, let's configure Tomcat to be managed by `systemctl`. Create a new systemd service file for Tomcat:

```
vim /etc/systemd/system/tomcat.service
```

Add the following content to the file:

```
[Unit]
Description=Tomcat
After=network.target

[Service]
Type=forking

User=tomcat
Group=tomcat

WorkingDirectory=/opt/tomcat

Environment=JAVA_HOME=/usr/lib/jvm/jre

Environment=CATALINA_HOME=/opt/tomcat
Environment=CATALINE_BASE=/opt/tomcat

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh

[Install]
WantedBy=multi-user.target
```

This defines how Tomcat should be started, stopped, and run as a service.

### **Step 9: Start and Enable Tomcat with Systemctl**

Now that the service is configured, reload system config changes

```
systemctl daemon-reload
```

- Start Tomcat using `systemctl`:

```
systemctl start tomcat
```

- Check the status of the Tomcat service:

```
systemctl status tomcat
```

- Finally, enable Tomcat to start on boot:

```
systemctl enable tomcat
```

### **Conclusion**

Congratulations! You've successfully installed Apache Tomcat and configured it as a systemd service on CentOS. Now you can manage Tomcat just like any other system service, making it easier to start, stop, and enable it on boot. This is particularly useful for production environments where consistency and automation are key.
