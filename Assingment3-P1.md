# ACIT2420 - Assignment 3 Part 1 | Tutorial 

**Andrew Huang | Set B | A01235926**

> #### Before you use this tutorial
>
> - You have created a droplet in DigitalOcean/
> - The droplet is a Debian 12 Server.
>

## Creating a Regular User

### Objective

The objective is to create a new regular user capable of performing administrative tasks, accessing the server via SSH, and having bash as the login shell.

### Step 1: Login to Debian Server

Start by logging in to your Debian Server on DigitalOcean using the Root user since it's the only user available at the moment. Open the terminal (MacOS) or PowerShell (Windows).

Let's start with opening up ```terminal (MacOS) or powershell (Windows)```

The command to log in to the Debian server is:

```ssh -i .ssh/do-key root@<IP-ADDRESS>```

- `-i`: This option specifies the private key file for authentication when connecting to a remote server.
- ```<IP-ADDRESS>```: Your Debian Server hosted on DigitalOcean.

### Step 2: Create user

The command to create a user is:

```sudo useradd -ms /bin/bash <user-name>```

- `-m` : This option creates the user's home directory if it doesn't exists.
- ``` -s /bin/bash``` : This option specifies the login shell for the new user as **/bin/bash**.

### Step 3: Give User Password

We should give our new user a password.

The command to give a user a password is:

``` sudo passwd <user-name>```

You'll be asked to type your password twice. For security, nothing you type will be show up on the screen.

### Step 4: Give User Elevated Privileges

We will be giving our user the **sudo** group. The sudo group will allow our user to use the command ```sudo```. This command will temporaily allow a user to perform tasks with elevated privileges. 

The command to give our user the sudo group is:

```usermod -aG sudo <user-name>```

- `-a` : This option is used with ```-G``` option to add the user to the specfic group without removing them from their current groups.
- `-G` : This option adds the user to group that is follwed by it. In this case, it would be adding the user to the sudo group.

### Step 5: Give User Access to Server via SSH

We will know switch to our user and make it possible for the user to connect to the server via SSH.

The command to switch the current user to our new user is:

```sudo su -l <user-name>```

- `su`: This command stands for "switch user" and allows us to change the active user account in the terminal session.
- `-l`: This option makes the system load all settings and environment variables as if the user had logged in directly.

You will need to enter the password the for the new user that was created in step 3.

Now, we will need to copy **.ssh directory** from the roots user's home directory to the new users home directory, including any files in the directory.

The command to copy the .ssh directory will be:

```sudo cp -r /root/.ssh  /home/<user-name>```

- `-r`: This option stands for recursive, allowing the copying of all directories and their contents.

Then, we need to change the ownership of the directory that we copied to be owned by the new user and the new users primary group.

The command to change the directory's ownership will be:

```sudo chown -r <user-name>:<user-name> /home/<user-name>/.ssh```

Now, we need to test that the new user can connect to the server. Do the following:

1. We want to exit our server by the command `exit`
2. Connect to server as the new user: ``` ssh -i .ssh/do-key <user-name>@<IP-ADDRESS>```

If you were able to log in as the new user, everything is working.

## Prevent Root User SSH Access 

### Objective

The objective will be preventing the root user from connecting to the server via SSH.

### Step 1: Login User with Administrative Privileges

We will need to login with a user that has administrative privileges because we will be modifying system-wide configuration files. Since the previous section we allowed our new user have administrative privileges, lets login to the server as the new user.

### Step 2: Edit SSH Configuartion File

We will want to open the `sshd_config` file in the `/etc/ssh/` directory with a text-editor like `vim`.

The command to open the configuration file in vim is:

```sudo vim /etc/ssh/sshd.config```

In the `sshd.config` file, use the search command `/` to look for the line `PermitRootLogin yes` and change it to `PermitRootLogin no`, in insert mode by pressing `i`. 

Save the file by pressing `esc` follwed by `wq`

### Step 3: Restart SSH Service

We will now want to restart the SSH service becasue we modifiy the ssh configuration files and want to apply the changes made.

The command to reload the SSH service is:

```sudo systemctl restart ssh.service```

Now try to connect to the debian server via ssh as the root user and it should deny the access.

## Create a Web Server with Nginx

### Objective

The objective is to create a simple web server that is up and running by using the nginx packages.

### Step 1: Install Nginx Packages

We will first install nginx packages. Nginx is a popular web server created by Igor Sysoev in 2005.

First we want to check for and fetches the latest information about available package updates with ```sudo apt update```

If it states there are updates available, we should install those updates with ```sudo apt upgrade```

Next, we will install the nginx package with ```sudo apt install nginx```

Lastly, if we ```sudo systemctl nginx``` you will see that it is enabled and running.

### Step 2: Setting up Simple Server

First we will create our web server folder, `my-site` in the ```/var/wwww``` directory.

The command to create our web server is:

```sudo mkdir -p /var/wwww/my-site```

- `-p`: This option creates the entire path to a directory, making sure all necessary parent directories are created, even if some of them are missing.
- `/var/wwww/`: We store our web server in this directory beacause it is where web content, such as website files and pages, is stored and served by web servers 

### Step 3: Setting up HTML page 

In our web server, we should probally attach a HTML page to it.

Lets create an `index.html` document that contains the code below in the `my-site` directory.

    <!DOCTYPE html>
    <html lang="en">
    <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>2420</title>
    <style>
        body {
            display: flex;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
        }
        h1 {
            text-align: center;
        }
    </style>
    </head>
    <body>
    <h1>Hello, World</h1>
    </body>
    </html>

This `index.html` is a simple HTML page that uses flexbox to center the content to be in the middle of our page.

The command to create an index.html in our web server is:

```sudo vim /var/www/my-site/index.html```

### Step 4: Creating a Server Block

The server block will manage incoming requests, determining how the server responds and serves web content.

We will create a new file in ```/etc/nginx/sites-available``` and paste the nginx config below into it. 

We create a file in the `sites-available` directory because it contains server configuration files that are available, but not active.

    # nginx config

    server {
        # Listen for incoming connections on port 80 (HTTP)
        listen 80;

        # Directory where website files are stored
        root /var/www/my-site;  

        # Default files to serve
        index index.html;  

        # Default for requests without specific server names
        server_name _; 

        location / {
            # First attempt to serve request as a file,
            # then as a directory, and if not found, display a 404 error.

            try_files $uri $uri/ =404;
        }
    }

The command to create a file in ```/etc/nginx/sites-available``` is:

```sudo vim /etc/nginx/sites-available/my-site.conf```

### Step 5: Create Symbolic Link

Currently our server configuartion file is in the `sites-available` directory which makes it available but not active. To activate a site, you will need to create a symbolic link in the `sites-enabled` directory.

We initally didn't create our server configuration file in the `sites-enabled` directly to make it possible to create multiple servers. By using symbolic links `ln -s` to create links and removing them `unlink`, we can easily activate or deactivate configurations.

The command to create a symbolic link to your new config file in ```/etc/nginx/sites-enabled``` is:

```sudo ln -s /etc/nginx/sites-available/my-site /etc/nginx/sites-enabled```

- `ln`: This command is used to create links
- `-s`: This option makes the link to be symbolic

### Step 6:  Unlink Default Configuration File

If you `ls` in ```/etc/nginx/sites-enabled``` directory, you will see a default configuration file that is linked.

In order to have our web server to run properly, we need to unlink the default configuration file.

The command to unlink the default configuration file is:

```sudo unlink default```

We should verify the Nginx configurations when we have a symbolic link in `sites-enabled`, execute the following command ```sudo nginx -t```.

If there are not errors displayed, everything is working.

### Step 7: Restart Nginx Service

We will now want to restart the nginx service becasue we modifiy the nginx configuration files and want to apply the changes made.

The command to reload the nginx service is:

```sudo systemctl restart nginx.service```

### Step 8: Test Web Server

Now lets test if everything we did above works!

You can test if the web server works in the following ways:

> The IP-ADDRESS is your Debian Server hosted on DigitalOcean.

1. Use the command ```curl <IP-ADDRESS>``` in terminal or powershell.
    - You should display the HTML structure of index.html

2. Open your preferred web browser and enter the `<IP-ADDRESS>` in the search bar.
    - You should be able to see **Hello World** centered in the page.

## Conclusion

In this tutorial, we have achieved the following:

- Create a new regular user:
    - Capable of performing administrative tasks
    - Accessing the server via SSH
    - Using bash as the login shell
- Prevent the root user from connecting to the server via SSH
- Install nginx
- Configure nginx to serve a sample website