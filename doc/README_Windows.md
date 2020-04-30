# Running Airflow on Windows 10 & WSL

Windows has come a long way in the last few years by catering to open source software and developers in an increasing capacity. Nothing illustrates this more than their development of WSL (Windows Subsystem for Linux), which allows you to install a Linux distribution on your PC alongside Windows without having to worry about VMs or containers. This is great for developers that work with tools that only run in Linux, such as Apache Airflow.

Airflow is top-level Apache project used for orchestrating workflows and data pipelines. It is quickly becoming a popular choice for organizations of all sizes and industries. Airflow is built in Python but contains some libraries that will only work in Linux, so workarounds using virtual machines or Docker are required for fully-functional usage. While both VMs and Docker are great options, this post will talk about setting up Airflow in WSL for very simple access to Airflow with little overhead.

## Installing WSL
WSL is a relatively new and feature of Windows. It's frequently updated and its functionality is rapidly improving. Instead of walking through all the steps to install here (since they may change), follow this doc from Microsoft for the latest steps. This post will focus on using Ubuntu for WSL, which you can download here.

Make sure you follow the link to initialize your new distro instance. This will create a Linux username and password. Remember this password- it is needed to use sudo going forward. Also run sudo apt update && sudo apt upgrade to make sure everything is up to date.

### Warning
Files in the Linux file system should not be accessed from Windows, as they can end up getting corrupted. Do not open or edit any files from processes running in Windows such as explorer, notepad, atom, or another IDE.

However, Linux can access files in the Windows file system. Because of this capability, I like to organize all my project files in the Windows file system. For example, you can create a folder in C:/Users/your_username/airflow and use it as your AIRFLOW_HOME directory. You can edit files (DAGs, plugins, etc.) in here with your favorite Windows editors (notepad, atom, PyCharm, etc.) and then use them with Airflow from WSL.

Future Windows updates may change this behavior and allow Linux files to be edited by Windows processes, so keep an eye out for that.

## Additional Setup
Once you have WSL installed, launch Ubuntu to land on the bash command prompt in your Linux home directory. If you type pwd (present working directory), you should see /home/user_name, indicating that you are in the Linux file system in your Linux home directory. Using ~ will reference this path.

Type `ls -a` to view the contents of this directory, including hidden objects. You’ll see `.bashrc`. This file is useful for changing settings such as environment variables that you want to apply to your bash sessions every time you start WSL.

The Windows file system is available to you in Linux and is located at `/mnt/c`. If you type `ls /mnt/c` you will see the contents of the `C:/` in windows. Note: you will see some permissions errors regarding symbolic links, you can ignore those.

It can be a bit burdensome to have to look in `/mnt/c` to get to your windows files. In fact, if you end up wanting to use Docker in WSL (which you probably will), this actually won’t work at all. So let’s change how the Windows file system is mounted in Linux.

Type sudo nano `/etc/wsl.conf`. This will open the Linux nano text editor for editing files. Change the file structure to match the following:

```
[automount]
root = /
options = "metadata"
```

Enter ctrl + s, ctrl + x to save the changes and exit nano. Now sign out, then back in to windows for these changes to take affect. Open Ubuntu and type ls / and you should see that c is now mounted on root / instead of /mnt/.

## Python 3
Confirm you have Python 3 installed with `python3 --version`.

Now let's install pip: `sudo apt update, sudo apt install python3-pip` should do the trick.

Run `pip3 --version` to make sure it's installed correctly.

## Installing Airflow
This step should be no different than installing Airflow in any normal Ubuntu environment. If you want, you can include other Airflow modules such as postrges or s3. Some of these may require dependencies to be installed on Ubuntu using sudo apt install [your_dependency].

Run `pip3 install apache-airflow`.

Now let’s set AIRFLOW_HOME (Airflow looks for this environment variable whenever Airflow CLI commands are run). When you run airflow init it will create all the Airflow stuff in this directory. As mentioned earlier, we want this to be in the Windows file system so you can edit all the files from Windows based tools.

Running nano `~/.bashrc` will open up the file mentioned earlier for setting environment variables in your bash session. On a new line add the following statement `export AIRFLOW_HOME=/c/Users/user_name/AirflowHome` statement replacing user_name with your actual windows home folder name. It should look something like this:

```
# ~/.bashrc: executed by bash(1) for non-login shells.
# see /usr/share/doc/bash/examples/startup-files (in the package bash-doc)
# for examples

export AIRFLOW_HOME=/c/Users/user_name/AirflowHome
Use ctrl + s, ctrl + x to save and exit nano. Now, anytime you open a bash session in Ubuntu, the AIRFLOW_HOME environment variable will be set to AirflowHome folder in your Windows home directory.
```

Close and reopen Ubuntu. airflow version should now show you the version of airflow you installed with out any errors and running airflow initdb should populate your AirflowHome folder with a clean setup for Airflow. You can run airflow webserver or airflow scheduler to start those services.

And that’s it- happy Airflowing!

Source: astronomer.io