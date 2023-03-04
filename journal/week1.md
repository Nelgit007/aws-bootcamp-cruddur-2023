# Week 1 — App Containerization

## Backend App Containerization


### Step 1


I ran Python using the following commands:


```sh
pip install -r requirements.txt
```


Then
```sh
cd backend-flask
export FRONTEND_URL="*"
export BACKEND_URL="*"
python3 -m flask run --host=0.0.0.0 --port=4567
cd ..
```


### Step 2

I ensured the port was open

I proceeded to append the url `/api/activities/home` to the link opened on my browser:

See outcome below;

[App Backend data]()

I then stopped the running application, then unset the Backend and Frontend URL's


### Step 3


I Created a docker file for the backend app @ `backend-flask/Dockerfile`

```dockerfile
FROM python:3.10-slim-buster

WORKDIR /backend-flask

COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

COPY . .

ENV FLASK_ENV=development

EXPOSE ${PORT}
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]
```


### Build Container For Backend Application

From the application dir, I ran the following command to build the container defined in the docker file above;

```sh
docker build -t  backend-flask ./backend-flask
```

RUN
```sh
docker run --rm -p 4567:4567 -it -e FRONTEND_URL='*' -e BACKEND_URL='*' backend-flask
```




