<h1>Setup Rentals Data in Cloud SQL</h1>
Overview
In this lab, you populate rentals data in Cloud SQL for the rentals recommendation engine to use.

What you learn
In this lab, you will:

Create Cloud SQL instance

Create database tables by importing .sql files from Cloud Storage

Populate the tables by importing .csv files from Cloud Storage

Allow access to Cloud SQL

Explore the rentals data using SQL statements from CloudShell

Introduction
In this lab, you populate rentals data in Cloud SQL for the rentals recommendation engine to use. The recommendations engine itself will run on Dataproc using Spark ML, and you will set that up in the next lab.

What you'll need
To complete this lab, you’ll need:

Access to a standard internet browser (Chrome browser recommended).

Time. Note the lab’s Completion time in Qwiklabs. This is an estimate of the time it should take to complete all steps. Plan your schedule so you have time to complete the lab. Once you start the lab, you will not be able to pause and return later (you begin at step 1 every time you start a lab).

The lab's Access time is how long your lab resources will be available. If you finish your lab with access time still available, you will be able to explore the Google Cloud Platform or work on any section of the lab that was marked "if you have time". Once the Access time runs out, your lab will end and all resources will terminate.

You DO NOT need a Google Cloud Platform account or project. An account, project and associated resources are provided to you as part of this lab.

If you already have your own GCP account, make sure you do not use it for this lab.

If your lab prompts you to log into the console, use only the student account provided to you by the lab. This prevents you from incurring charges for lab activities in your personal GCP account.

Start your lab
When you are ready, click Start Lab. You can track your lab’s progress with the status bar at the top of your screen.

Important What is happening during this time? Your lab is spinning up GCP resources for you behind the scenes, including an account, a project, resources within the project, and permission for you to control the resources needed to run the lab. This means that instead of spending time manually setting up a project and building resources from scratch as part of your lab, you can begin learning more quickly.
Find Your Lab’s GCP Username and Password
To access the resources and console for this lab, locate the Connection Details panel in Qwiklabs. Here you will find the account ID and password for the account you will use to log in to the Google Cloud Platform:

Open Google Console

If your lab provides other resource identifiers or connection-related information, it will appear on this panel as well.

Setup
Activate Google Cloud Shell
Google Cloud Shell provides command-line access to your GCP resources.

From the GCP Console click the Cloud Shell icon on the top right toolbar:

Cloud Shell Icon

Then click START CLOUD SHELL:

Start Cloud Shell

You can click START CLOUD SHELL immediately when the dialog comes up instead of waiting in the dialog until the Cloud Shell provisions.
It takes a few moments to provision and connects to the environment:

Cloud Shell Terminal

The Cloud Shell is a virtual machine loaded with all the development tools you’ll need. It offers a persistent 5GB home directory, and runs on the Google Cloud, greatly enhancing network performance and authentication.

Once connected to the cloud shell, you'll see that you are already authenticated and the project is set to your PROJECT_ID:

gcloud auth list

Output:

Credentialed accounts:
 - <myaccount>@<mydomain>.com (active)
Note: gcloud is the powerful and unified command-line tool for Google Cloud Platform. Full documentation is available on Google Cloud gcloud Overview. It comes pre-installed on Cloud Shell and supports tab-completion.
gcloud config list project

Output:

[core]
project = <PROJECT_ID>
Task 1: Access lab code
To explore the lab code in Cloud Shell:

In Cloud Shell, type:

git clone https://github.com/GoogleCloudPlatform/training-data-analyst

This downloads the code from github.

Navigate to the folder corresponding to this lab:

cd training-data-analyst/CPB100/lab3a

Examine the table creation file using less:

less cloudsql/table_creation.sql

The less command allows you to view the file (Press the spacebar to scroll down; the letter b to back up a page; the letter q to quit).

Fill out this information (the first line has been filled out for you):

Table Name	Columns
Accommodation	Id, title, location, price, rooms, rating, type
________________?	________________?
________________?	________________?
How do these relate to the rentals recommendation scenario? Fill the following blanks:

When a user rates a house (giving it four stars for example), an entry is added to the ________________ table.
General information about houses, such as the number of rooms they have and their average rating is stored in the ________________ table.
The job of the recommendation engine is to fill out the ________________ table for each user and house: this is the predicted rating of that house by that user.
Examine the data files using head:

head cloudsql/*.csv

The head command shows you the first few lines of each file.

Task 2: Create bucket
In the GCP Console, on the Navigation menu (8ab244f9cffa6198.png).

Click Storage.

Click Create Bucket.

For Name, enter your Project ID, then click Create. To find your Project ID, click the project in the top menu of the GCP Console and copy the value under ID for your selected project.

Note the name of your bucket. In the remainder of the lab, replace <BUCKET-NAME> with your unique bucket name.

Task 3: Stage .sql and .csv files into Cloud Storage
Stage the table definition and data files into Cloud Storage, so that you can later import them into Cloud SQL:

From Cloud Shell within the lab3a directory, type:

gsutil cp cloudsql/* gs://<BUCKET-NAME>/sql/

substituting the name of the bucket.

From the GCP console, navigate to Storage, then your bucket and verify that the .sql and .csv files now exist.

Task 4: Create Cloud SQL instance
In the GCP console, click SQL (in the Storage section).

Click Create instance.

Choose MySQL. Click Next if required. Click Configure MySQL Development or Choose Second Generation.

For Instance ID, type rentals.

ab1cdf08212ecadf.png

Scroll down and specify a root password. Before you forget, note down the root password.

Click Create to create the instance. It will take a minute or so for your Cloud SQL instance to be provisioned.

Task 5: Create tables
In Cloud SQL, click rentals to view instance information.

Click Import(on the top menu bar).

Click Browse. This will bring up a list of buckets. Click on the bucket you created, then navigate into sql and click table_creation.sql.

Click Select, then click Import.

Task 6: Populate tables
To import CSV files from Cloud Storage, from the GCP console page with the Cloud SQL instance details, click Import (top menu).

Click Browse, browse in the bucket you created to sql, then click accommodation.csv. Click Select.

For Database, select recommendation_spark.

For Table, type Accommodation.

c899adb7fdb39f17.png

Click Import.

Repeat the Import (steps 1 - 5) for rating.csv, but for Table, type Rating.

Task 7: Explore Cloud SQL
To explore Cloud SQL, you can use the mysql CLI. In Cloud Shell, type the following:

wget https://dl.google.com/cloudsql/cloud_sql_proxy.linux.amd64 -O cloud_sql_proxy
chmod +x cloud_sql_proxy

In the GCP Console, find the SQL Instance connection name in the Overview tab and copy it. In the command below, replace <INSTANCE_CONNECTION_NAME> with this value and run it.

./cloud_sql_proxy -instances=<INSTANCE_CONNECTION_NAME>=tcp:3306 &

Connect to the Cloud SQL instance using mysql:

mysql -u root -p --host 127.0.0.1

MySQL will prompt you for the root password. Enter the password when prompted.

In Cloud Shell, at the mysql prompt, type:

use recommendation_spark;

This sets the database in the mysql session.

View the list of tables you created. This will be helpful to prevent any typos in your query in step 4.

show tables;

Let's verify that the data was loaded.

select * from Rating;

Example output:

| 23     | 99     |      5 |
| 4      | 99     |      4 |
| 7      | 99     |      5 |
| 8      | 99     |      5 |
+--------+--------+--------+
1186 rows in set (0.03 sec)

Let's see if there is a great deal out there somewhere.

select * from Accommodation where type = 'castle' and price < 1500;

All the cheap castles are rated poorly.

You may exit the mysql prompt by typing exit.

End your lab
When you have completed your lab, click End Lab. Qwiklabs removes the resources you’ve used and cleans the account for you.

You will be given an opportunity to rate the lab experience. Select the applicable number of stars, type a comment, and then click Submit.

The number of stars indicates the following:

1 star = Very dissatisfied
2 stars = Dissatisfied
3 stars = Neutral
4 stars = Satisfied
5 stars = Very satisfied
You can close the dialog box if you don't want to provide feedback.

For feedback, suggestions, or corrections, please use the Support tab.

Manual Last Updated: November 20, 2018

Lab Last Tested: November 20, 2018

©2018 Google LLC All rights reserved. Google and the Google logo are trademarks of Google LLC. All other company and product names may be trademarks of the respective companies with which they are associated.

###=============================================================================================

<h1>Recommendations ML with Dataproc</h1>
Overview
In this lab, you carry out recommendations machine learning using Dataproc.

What you learn
In this lab, you will:

Launch Dataproc

Run SparkML jobs using Dataproc

Introduction
In this lab, you use Dataproc to train the recommendations machine learning model based on users' previous ratings. You then apply that model to create a list of recommendations for every user in the database.

In this lab, you will:

Launch Dataproc
Train and apply ML model written in PySpark to create product recommendations
Explore inserted rows in Cloud SQL
What you'll need
To complete this lab, you’ll need:

Access to a standard internet browser (Chrome browser recommended).

Time. Note the lab’s Completion time in Qwiklabs. This is an estimate of the time it should take to complete all steps. Plan your schedule so you have time to complete the lab. Once you start the lab, you will not be able to pause and return later (you begin at step 1 every time you start a lab).

The lab's Access time is how long your lab resources will be available. If you finish your lab with access time still available, you will be able to explore the Google Cloud Platform or work on any section of the lab that was marked "if you have time". Once the Access time runs out, your lab will end and all resources will terminate.

You DO NOT need a Google Cloud Platform account or project. An account, project and associated resources are provided to you as part of this lab.

If you already have your own GCP account, make sure you do not use it for this lab.

If your lab prompts you to log into the console, use only the student account provided to you by the lab. This prevents you from incurring charges for lab activities in your personal GCP account.

Start your lab
When you are ready, click Start Lab. You can track your lab’s progress with the status bar at the top of your screen.

Important What is happening during this time? Your lab is spinning up GCP resources for you behind the scenes, including an account, a project, resources within the project, and permission for you to control the resources needed to run the lab. This means that instead of spending time manually setting up a project and building resources from scratch as part of your lab, you can begin learning more quickly.
Find Your Lab’s GCP Username and Password
To access the resources and console for this lab, locate the Connection Details panel in Qwiklabs. Here you will find the account ID and password for the account you will use to log in to the Google Cloud Platform:

Open Google Console

If your lab provides other resource identifiers or connection-related information, it will appear on this panel as well.

Setup
Activate Google Cloud Shell
Google Cloud Shell provides command-line access to your GCP resources.

From the GCP Console click the Cloud Shell icon on the top right toolbar:

Cloud Shell Icon

Then click START CLOUD SHELL:

Start Cloud Shell

You can click START CLOUD SHELL immediately when the dialog comes up instead of waiting in the dialog until the Cloud Shell provisions.
It takes a few moments to provision and connects to the environment:

Cloud Shell Terminal

The Cloud Shell is a virtual machine loaded with all the development tools you’ll need. It offers a persistent 5GB home directory, and runs on the Google Cloud, greatly enhancing network performance and authentication.

Once connected to the cloud shell, you'll see that you are already authenticated and the project is set to your PROJECT_ID:

gcloud auth list

Output:

Credentialed accounts:
 - <myaccount>@<mydomain>.com (active)
Note: gcloud is the powerful and unified command-line tool for Google Cloud Platform. Full documentation is available on Google Cloud gcloud Overview. It comes pre-installed on Cloud Shell and supports tab-completion.
gcloud config list project

Output:

[core]
project = <PROJECT_ID>
Task 1: Create Assets
We start by cloning the code repository, creating a storage bucket in the GCP project and stage some files. These steps are similar to what you did in the previous lab.

In Cloud Shell, clone the repo using the following command:

git clone https://github.com/GoogleCloudPlatform/training-data-analyst

This downloads the code from github.

Navigate to the folder corresponding to this lab:

cd training-data-analyst/CPB100/lab3a

In the GCP Console, on the Navigation menu (mainmenu.png), click Storage.

Click Create bucket.

For Name, enter your Project ID, then click Create. To find your Project ID, click the project in the top menu of the GCP Console and copy the value under ID for your selected project.

Finally, stage the table definition and data files into Cloud Storage, so that you can later import them into Cloud SQL from Cloud Shell within the lab3a directory by typing the following, replacing <BUCKET-NAME> with the name of the bucket you just created:

gsutil cp cloudsql/* gs://<BUCKET-NAME>/sql/

From the GCP console, go to Storage, navigate to your bucket and verify that the .sql and .csv files now exist on Cloud Storage.

Task 2: Create Cloud SQL instance
To create Cloud SQL instance:

In the GCP Console, on the Navigation menu (mainmenu.png), click SQL (in the Storage section).

Click Create Instance.

Choose MySQL. Click Next if required. Click Configure MySQL Development or Choose Second Generation.

For Instance ID, type rentals.

ab1cdf08212ecadf.png

Scroll down and specify a root password. Before you forget, note down the root password (please don't do this in real-life!).

Scroll down and click Set Connectivity or Click Show configuration options first (if required), then click Authorize networks > +Add network.

ad40ddbc1cfc0a81.png

In Cloud Shell, make sure you're in the lab3a directory and find your IP address by typing:

bash ./find_my_ip.sh

In the Add Network dialog, enter an optional Name and enter the IP address output in the previous step. Click Done.

54d82efbc516bceb.png

Note: If you lose your Cloud Shell VM due to inactivity, you will have to reauthorize your new Cloud Shell VM with Cloud SQL. For your convenience, lab3a includes a script called authorize_cloudshell.sh that you can run.

Click Create to create the instance. It will take a minute or so for your Cloud SQL instance to be provisioned.

Note down the Primary IP address of your Cloud SQL instance (from the browser window).

Task 3: Create and populate tables
To import table definitions from Cloud Storage:

Click rentals to view details about your Cloud SQL instance.

Click Import.

Click Browse. This will bring up a list of buckets. Click on the bucket you created, then navigate into sql, click table_creation.sql, then click Select.

Click Import.

Next, to import CSV files from Cloud Storage, click Import.

Click Browse, navigate into sql, click accommodation.csv, then click Select.

Fill out the rest of the dialog as follows:

For Database, select recommendation_spark

For Table, type Accommodation

c899adb7fdb39f17.png

Click Import.

Repeat the Import process (steps 5 - 8) for rating.csv, but for Table, type Rating.

Task 4: Launch Dataproc
To launch Dataproc and configure it so that each of the machines in the cluster can access Cloud SQL:

In the GCP Console, on the Navigation menu (mainmenu.png), click SQL and note the region of your Cloud SQL instance:

fc8f254ae64a75b4.png

In the above snapshot, the region is us-central1.

In the GCP Console, on the Navigation menu (mainmenu.png), click Dataproc and click Enable API if prompted. Once enabled, click Create cluster.

Leave the Region as it is i.e. global, change the Zone to be in the same region as your Cloud SQL instance. This will minimize network latency between the cluster and the database.

For Master node, for Machine type, select 2 vCPU (n1-standard-2).

For Worker nodes, for Machine type, select 2 vCPU (n1-standard-2).

Leave all other values with their default and click Create. It will take 1-2 minutes to provision your cluster.

Note the Name, Zone and Total worker nodes in your cluster.

In Cloud Shell, navigate to the folder corresponding to this lab and authorize all the Dataproc nodes to be able to access your Cloud SQL instance, replacing <Cluster-Name>, <Zone>, and <Total-Worker-Nodes> with the values you noted in the previous step:

cd ~/training-data-analyst/CPB100/lab3b
bash authorize_dataproc.sh <Cluster-Name> <Zone> <Total-Worker-Nodes>

When prompted, type Y, then Enter to continue.

Task 5: Run ML model
To create a trained model and apply it to all the users in the system:

Edit the model training file using nano:

nano sparkml/train_and_apply.py

Change the fields marked #CHANGE at the top of the file (scroll down using the down arrow key) to match your Cloud SQL setup (see earlier parts of this lab where you noted these down), and save the file using Ctrl+O then press Enter, and then press Ctrl+X to exit from the file.

Copy this file to your Cloud Storage bucket using:

gsutil cp sparkml/tr*.py gs://<bucket-name>/

In the Dataproc console, click Jobs.

8508ce301ff584c3.png

Click Submit job.

For Job type, select PySpark and for Main python file, specify the location of the Python file you uploaded to your bucket.

e15bafe2c29956b5.png

gs://<bucket-name>/train_and_apply.py

Click Submit and wait for the job Status to change from Running (this will take up to 5 minutes) to Succeeded.

af55e2a91617b1d5.png

If the job Failed, please troubleshoot using the logs and fix the errors. You may need to re-upload the changed Python file to Cloud Storage and clone the failed job to resubmit.

Task 6: Explore inserted rows
In the GCP Console, on the Navigation menu (mainmenu.png), click SQL (in the Storage section).

Click rentals to view details related to your Cloud SQL instance.

Under Connect to this instance section, click Connect using Cloud Shell. This will start new Cloudshell tab. In Cloudshell tab press Enter.

It will take few minutes to whitelist your IP for incoming connection.

When prompted, type the root password you configured, then Enter.

At the mysql prompt, type:

use recommendation_spark;

This sets the database in the mysql session.

Find the recommendations for some user:

select r.userid, r.accoid, r.prediction, a.title, a.location, a.price, a.rooms, a.rating, a.type from Recommendation as r, Accommodation as a where r.accoid = a.id and r.userid = 10;

These are the five accommodations that we would recommend to her. Note that the quality of the recommendations are not great because our dataset was so small (note that the predicted ratings are not very high). Still, this lab illustrates the process you'd go through to create product recommendations.

End your lab
When you have completed your lab, click End Lab. Qwiklabs removes the resources you’ve used and cleans the account for you.

You will be given an opportunity to rate the lab experience. Select the applicable number of stars, type a comment, and then click Submit.

The number of stars indicates the following:

1 star = Very dissatisfied
2 stars = Dissatisfied
3 stars = Neutral
4 stars = Satisfied
5 stars = Very satisfied
You can close the dialog box if you don't want to provide feedback.

For feedback, suggestions, or corrections, please use the Support tab.

Manual Last Updated: November 20, 2018
Lab Last Tested: November 20, 2018
©2018 Google LLC All rights reserved. Google and the Google logo are trademarks of Google LLC. All other company and product names may be trademarks of the respective companies with which they are associated.

###=============================================================================================