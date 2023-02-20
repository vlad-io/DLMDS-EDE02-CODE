# Purpose

This docker-compose consists of 3 images that follow the steps in the pipeline

## Pipelines steps:

A. Injestor: download the Kaggle data into `injestor/data-in` folder
B. Processor: unpack the information, from `injestor/data-in` into `processor/data-out` 
C. ML Frontend: provides a jupyter notebook environment to analyse the data in `ml_frontend/data-in`, which is mapped to the `processor/data-out`

## Setup:

1. Clone the project with gitclone https://github.com/vlad-io/dlmds-ede02-code

The folder structure would follow:

![project folder structure](/assets/folder-structure.png)

1. Ensure that you have a Kaggle sign-in from https://www.kaggle.com/. The free account is necessary to retrieve the dataset.
2. Retrieve the API token from https://www.kaggle.com/account. The token file will contain the username and key.
3. Update Kaggle username and key as your environmental variables. For example one can use "dlmds-ede02-code/.env" file:
        `KAGGLE_NAME=yourkaggleusername
        KAGGLE_KEY=yourkaggleapikey`
4. In terminal, navigate to the project folder: 
        `cd dlmds-ede02-code`
5. Build the images
        `docker-compose build`
![docker-compose up expected output](/assets/docker-compose-build.png)

If the KAGGLE environmental variables were not setup correctly the following will show
![docker-compose up expected output](/assets/docker-compose-build-no-env.png)

6. Run the containers
        `docker-compose up`
![docker-compose up expected output](/assets/docker-compose-up.png)

As the docker composer launches the containers and one of the lines in the terminal output will print an address to the jupyter notebook. It should be similar to `http://127.0.0.1:8888/lab?token=....` Navigate to that link in the browser. The processed data (the csv files) will be available in the `/data-in` folder

![jupyter access link example](/assets/jupyter-link.png)

7. To shut down, stop the containers in Docker Desktop (if installed) or Ctrl+C in the terminal where the `docker-compose up` command was executed.

# Infrastructure as code

The Docker image details

A. Injestor

The docker image that retrieves a dataset from Kaggle. It can be used independently from the docker-compose, if a 'data-in' volume is provided in the `docker run` command

The `./injestor` folder that contains:

a. Dockerfile with:

- python base image

- installation additional kaggle library that is needed to retrieve the dataset

b. crontab file 

- Specifies the schedules and the kaggle download command

- First line (@reboot) specifies that the download should run immediately when the container starts

- Second line label specifies the frequency schedule:

Downloads hourly (as per "@hourly" label). If a different schedule is needed the "@hourly" label should be updated. For a posible set of schedule one can use the following examples:

    -- every minute: */1 * * * *
    -- every day: 0 1 * * *
    -- once a month: 0 0 1 * *
    -- every quarter: 0 0 1 */3 *

- Second line is the Kaggle download command as explained in https://www.kaggle.com/docs/api#interacting-with-datasets

- If a different dataset is needed the 'sudalairajkumar/novel-corona-virus-2019-dataset' should be changed to the desired label
- The destination folder is the persistent volume that is accessible from the next stage (the Processor) in pipeline.

- Current dataset is "Novel corona virus 2019 dataset", that can be found at https://www.kaggle.com/datasets/sudalairajkumar/novel-corona-virus-2019-dataset
- Currently the dataset has several million records which include geographic and time series data.
- When container is launched it pulls the dataset, and creates a schedule (crond) that attempts to refresh the data.
- The current currently set to hourly, because the dataset is expected to be updated daily, but unknown at which time.
- The Dockerfile contains the Kaggle access usename and key, which are necessary to access the dataset



B. Processor

The docker image that unpacking the injested data on schedule.

- Current schedule is set to every 5 minutes, just in case the user makes a mistake and updates the source data.

- Currently uses the container's standard command 'unzip' unpack the dataset from /data-in into /data-out.

- Uses a python image with the view to expand the processing by using python scripts.

The `./processor` folder contains:

a. Dockerfile:

python base image. Although a smaller Docker image can be used (without python) that has the `unzip` command, for consistency the same image as for the `./injestor` is reused.

b. crontab file 

> Specifies the schedules for the kaggle download command
> First line (@reboot) specifies that the download should run immediately when the container starts
> Second line label specifies the frequency schedule:

C. Machine Learning (ML) frontend

- Uses the unamended jupyter/pyspark-notebook image from the Docker hub. 

- Exposes port 8888 to access the notebook 

- As the docker composer launches the container one of the lines in the terminal output will print an address to the jupyter notebook. It should be similar to http://127.0.0.1:8888/lab?token=1f0cf2e65e04afc5bc03f0bc63b9cf87a9a980acfec13c2b. Navigate to that link in the browser. 

- The processed data will be available in the '/data-in' folder

- The notebooks can be saved in the work folder

- Navigating to http://127.0.0.1:8888 will open the correct page, but request the token that is only available from the terminal output (as above)
