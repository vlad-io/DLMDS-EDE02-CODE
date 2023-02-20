# Purpose

This docker-compose consists of 3 images that follow the steps in the batch processing pipeline.

## Pipelines steps:

1. Injestor: download the Kaggle data into `injestor/data-in` folder.
2. Processor: unpack the information, from `injestor/data-in` into `processor/data-out`.
3. Machine Learning (ML) frontend: provides a jupyter notebook environment to analyse the data in `processor/data-out`, which is visible as `ml_frontend/data-in`.

## Dataset
   - Current dataset is "Novel corona virus 2019 dataset", that can be found at https://www.kaggle.com/datasets/sudalairajkumar/novel-corona-virus-2019-dataset
   - Currently the dataset has several million records which include geographic and time series data.
   - If a different dataset is needed:
     - Update the 'sudalairajkumar/novel-corona-virus-2019-dataset' in the `./injestor/crontab' to the desired label obtained from kaggle
     - (Re)run `docker-compose build`
     - Run `docker-compose up' to launch the application
   - When container is launched it makes an initial pull of the dataset, and creates a schedule (crond) that refreshes the data.
   - The current refresh rate currently set to hourly, because the dataset is expected to be updated daily, but unknown at which time.

## Setup and deployment:

1. Clone the project with `gitclone https://github.com/vlad-io/DLMDS-EDE02-CODE.git`

   The command will create the project folder that will have the following structure:

   ![project folder structure](/assets/folder-structure.png)

2. Ensure that you have a Kaggle sign-in from https://www.kaggle.com/. The free account is necessary to retrieve the dataset.
3. Retrieve the API token from https://www.kaggle.com/account. The token file will contain the username and key.
4. Update Kaggle username and key as your environmental variables. For example one can use `dlmds-ede02-code/.env` file to store the credentials without exposing the sensitive information to public through the git repository:
   ```
   KAGGLE_NAME=your-kaggle-username
   KAGGLE_KEY=your-kaggle-api-key
   ```
5. In terminal, navigate to the project folder: 
   `cd dlmds-ede02-code`
6. Build the images
   `docker-compose build`
   ![docker-compose up expected output](/assets/docker-compose-build.png)

   If the KAGGLE environmental variables were not setup correctly the following WARNINGS will show
   ![docker-compose up expected output](/assets/docker-compose-build-no-env.png)

7. Run the containers
   `docker-compose up`
   ![docker-compose up expected output](/assets/docker-compose-up.png)

   As the docker composer launches the containers, one of the lines in the terminal output will print an address to the jupyter notebook. It should be similar to `http://127.0.0.1:8888/lab?token=....` 

   ![jupyter access link example](/assets/jupyter-link.png)

8. Navigate to that link in the browser. 
   ![jupyter notebook](/assets/jupyter-notebook-1.png)

9. The processed data (the csv files) will be available in the `/data-in` folder.

10. In needed save the workbooks in the `/work` folder.

11. To shut down
    - Stop the containers in Docker Desktop (if installed) 
    ![Docker desktop](/assets/docker-desktop.png)
    - or `Ctrl+C` in the terminal where the `docker-compose up` command is running.
    ![Ctrl+C in terminal](/assets/stop-ctrl-c.png)

# Infrastructure as code

The Docker image details

## Injestor

The docker image that retrieves a dataset from Kaggle. It can be used independently from the docker-compose, if a 'data-in' volume is provided in the `docker run` command

The `./injestor` folder contains:

a. Dockerfile with:

   - python base image

   - installation of the additional kaggle library that is needed to retrieve the dataset

b. crontab file 

   - Specifies the schedules and the kaggle download command

   - First line (@reboot) specifies that the download should run immediately when the container starts

   - Second line label specifies the frequency schedule:

     Downloads hourly (as per "@hourly" label). If a different schedule is needed the "@hourly" label should be updated. For a posible set of schedule one can use the following examples:

     - every minute: */1 * * * *
     - every day: 0 1 * * *
     - once a month: 0 0 1 * *
     - every quarter: 0 0 1 */3 *

   - Second line is the Kaggle download command as explained in https://www.kaggle.com/docs/api#interacting-with-datasets

   - The destination folder is the persistent volume that is accessible from the next stage (the Processor) in pipeline.

## Processor

The docker image that unpacking the injested data on schedule.

- Current schedule is set to every 5 minutes, just in case the user makes a mistake and updates the source data.

- Currently uses the container's standard command 'unzip' unpack the dataset from /data-in into /data-out.

- Uses a python image with the view to expand the processing by using python scripts.

The `./processor` folder contains:

a. Dockerfile:

python base image. Although a smaller Docker image can be used (without python) that has the `unzip` command, for consistency the same image as for the `./injestor` is reused.

b. crontab file 

   - Specifies the schedules for the kaggle download command
   - First line (@reboot) specifies that the download should run immediately when the container starts
   - Second line label specifies the frequency schedule:

## Machine Learning (ML) frontend

   - Uses the unamended jupyter/pyspark-notebook image from the Docker hub. 

   - Exposes port 8888 to access the notebook 

   - Navigating to http://127.0.0.1:8888 will open the correct page, but request the token that is only available from the terminal output of the `docker-compose up` command (as explained above)
