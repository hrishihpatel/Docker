
### Contents

[Install Docker for Windows](#install-docker-for-windows)<br/>
[Setup Docker for Windows Containers](#setup-docker-for-windows-containers)<br/>
[Running a SQL Server container](#running-a-sql-server-container)<br />
[Obtain IP Address of Docker Image](#obtain-ip-address)<br />
[Connect using Microsoft Management Studio)(#connect-with-SQLMgmtStudio)<br />


<a name=install-docker-for-windows></a>
# Install Docker

You can [download Docker here](https://www.docker.com/products/docker-desktop). Make sure to use the latest version of the Docker.

- Double-click Docker for Windows Installer.exe to run the installer.
- Follow the install wizard to accept the license, authorize the installer, and proceed with the install.
- You are asked to authorize Docker.app with your system password during the install process. Privileged access is needed to install networking components, links to the Docker apps, and manage the Hyper-V VMs.
- Click Finish on the setup complete dialog to launch Docker.

Additionally I would recommend to install Kitematic, a simple and powerful graphical user interface to manage your Docker environment.

To run SQL Server inside of a Docker container, you must have at least 3.25 GB allocated (Docker defaults to 2 GB, and if you leave this setting, SQL Server won't run). So click on the whale, go to Preferences, and increase the memory (I recommend 4 GB at a minimum, but if you can afford more, go for it). Then hit Apply & Restart:

<a name=setup-docker-for-windows-containers></a>
# Setup Docker for Windows Containers
After installing Docker on your machine Docker will start automatically in the background. As per default, Docker runs with Linux containers and you have to switch to Windows containers first. 

- Right click on your Docker icon in the taskbar and select “Switch to Windows containers”.

<a name=running-a-sql-server-container></a>
# Running a SQL Server container

At this point you should have installed the docker so let's jump right into it. Open Terminal window and pull the latest version of MS SQL from docker store. 

``` docker pull microsoft/mssql-server-windows ```

After your image download has been completed you can start your SQL Server image with the following command:

```
docker run -d –p <Port> –name <FriendlyName> -e sa_password=<Password> -e ACCEPT_EULA=Y microsoft/mssql-server-windows-developer:2017-latest
```
For Example

```
docker run -d -p 1433:1433 -v C:/temp/:C:/temp/ -e sa_password=Super@Password@123! -e ACCEPT_EULA=Y microsoft/mssql-server-windows
```

- -p **HostPort:containerPort** is for port-mapping a container network port to a host port.

- -v **HostPath:containerPath** is for mounting a folder from the host inside the container.

This can be used for saving database outside of the container.

- -**it** can be used to show the verbose output of the SQL startup script.


That is all you need to do, you can see if you docker container is running by running "docker ps -a".

If you're not a Terminal fan, i would recommend installing [Kitematic](https://kitematic.com/), a visual representation of Docker Container.

<a name=obtain-ip-address></a>
# Obtain the IP address of your Docker image

```
docker inspect --format '{{.NetworkSettings.Networks.nat.IPAddress}}' <ImageName>
```
For Example
```
docker inspect --format '{{.NetworkSettings.Networks.nat.IPAddress}}' mssql
```
<a name=connect-with-SQLMgmtStudio></a>
# Connect with SQL Server Management Studio

Based on the IP address of your docker image, you may connect to your MSSQL using the IP and port 1433

# Backup And Restore Database

Backup (Make sure to Backup prior to shutting down the container)
```
USE DockerLab;  
GO  
BACKUP DATABASE DockerLab  
TO DISK = '/Documents/DockerLab.bak'
GO

```

Restore 

Next, let's prepare to restore a backup file from the host. I happened to have an old Northwind backup in my Documents folder, so I can check the logical file names from RESTORE FILELISTONLY, and see where I need to move the data and log files based on the results of sp_helpfile:

```
RESTORE FILELISTONLY   
FROM DISK = N'/Documents/DockerLab.bak';
EXEC sp_helpfile;
```

Now I can attempt to restore the database, and verify that it was successful by querying a catalog view. Note that, in spite of the output of sp_helpfile and catalog views like sys.database_files, I can use proper Linux paths like /var/opt/... instead of pseudo-Windows paths like C:\var\opt\... output:

```
RESTORE DATABASE DockerLab
FROM DISK = N'/Documents/DockerLab.bak'
WITH MOVE N'DockerLab' TO N'/var/opt/mssql/data/dl.mdf',
     MOVE N'DockerLab_log' TO N'/var/opt/mssql/data/dl.ldf',
     REPLACE, STATS = 20;
GO
SELECT name from DockerLab.sys.tables;
```
