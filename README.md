# pipelining
Executive master Intelligence Artificielle et Science des données, cours : Données sur le cloud

[Link to the course slides](https://docs.google.com/presentation/d/1y02d8T-svCBt_LePXXv-q00uTskQCgrmHtUv6nTxF-Q/edit?usp=sharing)


## Lab Session 

The aim of the lab session is to:
- create an AWS EC2 Amazon Linux 2 instance,
- install Apache Airflow on the EC2 instance,
- write an Apache Airflow DAG, that downloads data from a cloud storage, trains a sklearn model, and uploads a pickle of the model on S3. The pipeline should run hourly, the model pickle should be stored in a folder on S3 prefixed by the execution date of the pipeline in the format `YYYY/MM/DD/HH`.


### Install Airflow on AWS EC2 Amazon Linux 2
Once you've create the EC2 instance according to the GIF in the [slides](https://docs.google.com/presentation/d/1y02d8T-svCBt_LePXXv-q00uTskQCgrmHtUv6nTxF-Q/edit?usp=sharing).

The instance is running:
![Running instances](https://github.com/faouzelfassi/pipelining/blob/master/doc/ec2_home.png?raw=true)

Now, let's connect to it through ssh:
![Browser-based SSH ](https://github.com/faouzelfassi/pipelining/blob/master/doc/ec2_ssh.png?raw=true)

Once connected we are going to install Airflow, just as you did on your personal computers.

1. First of all, let's install pip, to do so, `curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py` and `python get-pip.py`
1. Then, let's install python3, run the following commands:
	1. To install python3, run: `sudo yum install python3 -y`
	1. Now, let's create a virtual environment by doing: `python3 -m venv my_app/env`
	1. Let's activate the virtual env.: `source ~/my_app/env/bin/activate`
	1. Upgrade pip: `pip install pip --upgrade`
	1. Install boto3: `pip install boto3`
	1. Now, make sure `python` displays Python 3.x.x.
	1. Now install development toolkit, `sudo yum groupinstall "Development Tools"`
	1. Then install python3 development tools, `sudo yum install python3-devel`
1. Finally, install Apache Airflow, `pip3 install apache-airflow`
1. Once the installation is successful, initialize Airflow DB `airflow initdb`
1. Eventually, run the webserver and the scheduler **as deamons** (thanks to `&`): `airflow webserver &` hit enter, then  `airflow scheduler &` hit enter. **Everytime you restart your EC2 node you'll have to restart those two services to user Airflow (don't forget to be in your virtual environment `source ~/my_app/env/bin/activate` ⬆)**.
	* Note: In real life we use a service manager, like [systemd](https://doc.ubuntu-fr.org/systemd) to handle start/stop of services on your behalf, you can try configuring one by following [this guide](https://doc.ubuntu-fr.org/creer_un_service_avec_systemd).
1. Verify the installation by opening a browser in your computer and copy/paste the public IP of the EC2 instance followed by `:8080` in the address bar.
1. 🎉 **Congratulations you just installed Apache Airflow in the cloud**

### Continuous deployment
In order to gain productivity during the development of your pipeline, we want to synchronize your local work with the remote instance, to do so - in the simplest possible way - we are going to use a [crontab](https://doc.ubuntu-fr.org/cron).
A crontab is a simple mechanism that allows us to execute a bash command on a specific schedule.

We will use this mechanism on the remote instance to synchronize your master branch.

To do so, you'll need to create a git accound if needed, and a public git repository, it will host python Airflow DAGs, libraries and dependencies.

#### Instructions
1. Create a public git repository containing a `README.md` file and a `dags` folder.

**On your local environment:**
1. Clone the git repository in the same context than your airflow installation, not necessarily the same location, but somewhere nearby: `git clone https://github.com/{user_name}/{repository_name}`

**On the remote instance:**
1. Set a cron using `EDITOR=nano crontab -e` to pull your public git repository's master branch every minute, the command should be: `* * * * * cd /home/ec2-user/iasd; git pull origin master`, `iasd` should be modified to match your git repository name. To exit the nano editor: `ctrl+x` > `"Yes"` > `Enter`.
1. Make sure the git repository is correctly pulled by pushing a dummy commit in master.


🎉 **Congratulations you just synchronized your local work with a remote instance through GitHub**

### Creating your first ML pipeline on Airflow

We are going to train a Random Forest model based on [this blog post (part 1)](https://stackabuse.com/random-forest-algorithm-with-python-and-scikit-learn/). You can of course try to wrap your own python model or a Spark job you already have (contact me for the latter).

#### Generating an AWS programatic access
Now you've Airflow all setup up and running let's make it communicate with AWS, to do so we have to generate a new programatic user dedicated to Airflow on the AWS console.

1. Go to the AWS console website.,
1. In services, go to "Identity and Access Management" (IAM),
1. Click on the menu item "Access Management" in the left pan,
1. Then, click on "Users", then click on "Add new user",
1. Add a user name then click on "Programmatic access", click on "Next: Permissions",
1. Now, click on "Attach existing policies directly" and select the following: `AmazonS3FullAccess`, `AmazonElasticMapReduceFullAccess`, `AmazonEC2FullAccess`, `AmazonSageMakerFullAccess`
1. Then, click on "Next: Tags", "Next: Review" and finally "Create user". 
1. Now you have access to the Access Key and Secret access key, copy/paste them on your computer, we will set them in the Airflow We interface in the tab Admin > Connections, to give the permissions to Airflow for managing AWS services on your behalf.

You're all set, now let's deep dive in the pipeline creation.

##### Airflow DAG
The idea is to wrap an already made model in a `PythonOperator` that takes the downloaded data file as a parameter, and the model output location as a parameter.


1. First of all, [download this csv](https://drive.google.com/file/d/1mVmGNx6cbfvRHC_DvF12ZL3wGLSHD9f_/view) and put it in S3.
1. Have a look at [S3Hook](https://airflow.apache.org/docs/stable/_modules/airflow/hooks/S3_hook.html) for handling the communication with AWS S3.
1. Here is an example of a [PythonOperator](https://airflow.apache.org/docs/stable/howto/operator/python.html) in use.
1. Also, have a look at [Airflow Macros](https://airflow.apache.org/docs/stable/macros-ref.html#macros), handy for getting some variables around the execution of the DAG. Useful for outputing in a folder prefixed by a date representing the execution date of the pipeline run.


##### Note: pickle
Saving a model to disk is as simple as:

```python
import pickle

# save the model to disk
filename = 'finalized_model.sav'
pickle.dump(model, open(filename, 'wb'))
```

Symmetrically, loading a model from disk:

```python
# load the model from disk
filename = 'finalized_model.sav'
loaded_model = pickle.load(open(filename, 'rb'))
```

