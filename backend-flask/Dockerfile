FROM python:3.10-slim-buster

#Inside the contaier
# This command makes a new dir inside container
WORKDIR /backend-flask

#Moves requirements.txt (Libraries to run app) from outside the container to inside the conateiner
COPY requirements.txt requirements.txt

#Runs command inside container
#Thus will install python libraries/dependencies needed to run the app
RUN pip3 install -r requirements.txt

#This command copies everything in /Backend-flask dir outside the container to
# the /Backend-flask dir inside the container
COPY . .

#Sets ENV Vars inside the container
ENV FLASK_ENV=development

EXPOSE ${PORT}

#These cmd runs the flask web-app frmework
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]