# https://www.linkedin.com/pulse/how-schedule-python-application-docker-container-srinivas-reddy/
# processor
# Python Base Image
FROM python:3.9-alpine

# Creating Working 
WORKDIR /py_cronjob

# Copying the crontab file 
COPY crontab /etc/cron.d/crontab

# Copy the each file from project project to py_cronjob in docker container
ADD . /py_cronjob

#run the crontab file
RUN crontab /etc/cron.d/crontab

# # Executing crontab command
#run the cron process
CMD ["/usr/sbin/crond", "-f"]