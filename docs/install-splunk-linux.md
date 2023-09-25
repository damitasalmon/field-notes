---
tags: 
- splunk
- linux
---

# Download and Install Splunk Linux

For my Splunk labs I'm using Ubuntu on VirtualBox. 

You can download Splunk from the website. For Ubuntu, you can download the debian package. For RedHat or CentOS, you may want to use the .rpm package. As it begins to download, cancel. A box will appear with the link to download via the command line.

![Download link for Splunk](./assets/images/splunk-labs/download-splunk.png)

![View download link for Splunk](./assets/images/splunk-labs/download-splunk-2.png)

On Ubuntu this will download the tarball. Move the tarball to the `/opt` folder:

``` sh
sudo mv \<splunk tarball\>.tgz /opt`
cd /opt
sudo tar -xvzf \<splunk tarball\>.tgz
```

There should be a Splunk folder extracted to the /opt folder. To start, first navigate to the `bin` folder:

``` sh
cd splunk/bin/
sudo ./splunk start --accept-license
```

Create Admin username and password. Splunk is served on port 8000 by default. 

