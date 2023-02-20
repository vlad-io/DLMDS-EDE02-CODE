# https://www.linkedin.com/pulse/how-schedule-python-application-docker-container-srinivas-reddy/
# injestor
# Python Base Image
FROM python:3.9-alpine

# Creating Working 
WORKDIR /py_cronjob

#install kaggle library that is needed to download the data files
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir kaggle

# Copying the crontab file 
COPY crontab /etc/cron.d/crontab

# Copy the each file from project project to py_cronjob in docker container
ADD . /py_cronjob

#run the crontab file
RUN crontab /etc/cron.d/crontab

# download for the first time
ARG KAGGLE_USERNAME
ARG KAGGLE_KEY
ENV KAGGLE_USERNAME ${KAGGLE_USERNAME}
ENV KAGGLE_KEY ${KAGGLE_KEY}

# # Executing crontab command
#run the cron process
CMD ["/usr/sbin/crond", "-f"]