# Install Docker

You can [download Docker here](https://docs.docker.com/v17.12/docker-for-mac/install/). Make sure to use the latest version of the Docker.

Run the DMG. It's pretty simple; you just drag Docker.app into your Applications folder (there's even a helpful visual guide). Then go to the Applications folder and double-click Docker.app. You'll have a series of prompts, including one to enter your administrator password to let the application fiddle with your network settings. Then Docker will be up in your menubar – it’s a whale, but definitely not the fail whale.

To run SQL Server inside of a Docker container, you must have at least 3.25 GB allocated (Docker defaults to 2 GB, and if you leave this setting, SQL Server won't run). So click on the whale, go to Preferences, and increase the memory (I recommend 4 GB at a minimum, but if you can afford more, go for it). Then hit Apply & Restart:

# Running a SQL Server container

At this point you should have installed the docker so let's jump right into it. Open Terminal window and pull the latest version of MS SQL from docker store. 

``` docker pull microsoft/mssql-server-linux ```

You can determine success by looking for the following output:

```
Using default tag: latest
latest: Pulling from microsoft/mssql-server-linux
…
Digest: sha256:238…
Status: Downloaded newer image for microsoft/mssql-server-linux:latest
```
Now that we have the latest image of MSSQL server, let's spin up the container and connect using local MSSQL client. Run the following command and we'll cover what each parameter in the below code means.

Repalce USERID with your current path.

```
docker run -v /Users/<USERID>/Documents:/Documents -i -e ACCEPT_EULA=Y -e SA_PASSWORD=Super@Password@123! -p 1433:1433 -d microsoft/mssql-server-linux

```

- -v to mount a volume, so you can attach, restore, etc. using files from the host. Note that you have to specify -v first in order to avoid errors.
- -i to specify interactive (which really means "attach STDIN and keep it open"). This seemed to eliminate at least one of the connection roadblocks I faced in the early going.
- -e (twice) to specify environment variables ACCEPT_EULA and SA_PASSWORD.
For the EULA, this is pretty standard. You need to agree to the terms (even if this actually encourages you not to read them).

About the password: Note that the sa password needs to be relatively complex. I suspect it's based on the default AD implementation, but I don't know that the actual complexity rules are documented. If your password is not complex enough, as Jemeriah noted, the container will just vanish without warning. Also, I recommend avoiding special characters like $, which require cumbersome escaping (\$); sadly, this is where I spent quite easily the second-most amount of troubleshooting time.

- -p to let the host see the ports published by the container. I'll use 1433 here, because I had problems connecting on other ports (I haven't fully investigated that yet).
- -d to run the container in the background.
The last argument is unnamed and tells Docker which image to use. (You can see the list of available images you have with docker images.)


That is all you need to do, you can see if you docker container is running by running "docker ps -a".

# Connect using Azure Data Studio (AKA SQL Management Studio)

You can [download Azure Data Studio here](https://github.com/Microsoft/azuredatastudio)

# Backup And Restore Database

Backup
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

# References
[Sntryone](https://blogs.sentryone.com/aaronbertrand/vs-code-mac-sql-linux-docker/)
