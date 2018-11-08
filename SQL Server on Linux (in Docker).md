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
