# Django-React-Docker

This repo is a basic structure for projects where Django is used for backend, React for fronted, Postgres as DB and Docker for build. In this repo I've created 2 containers, one for backend and the other one for frontend and also setting up the Django for delivering APIs.

### Step 1: Create an empty project directory and inside it, 2 more folders: backend and frontend.

### Step 2: Inside 'backend' folder create Dockerfile
      ```dockerfile
      FROM python:3.8.3-alpine
      ENV PYTHONUNBUFFERED 1
      RUN mkdir /code
      WORKDIR /code
      COPY requirements.txt /code/
      RUN \
       apk add --no-cache postgresql-libs && \
       apk add --no-cache --virtual .build-deps gcc musl-dev postgresql-dev && \
       python3 -m pip install -r requirements.txt --no-cache-dir && \
       apk --purge del .build-deps
      COPY . /code/
      '''
### Step 3: Inside 'backend' folder create requirements.txt and add:

      Django==2.2.14
      psycopg2-binary>=2.7.4
      djangorestframework==3.11.0
      Pygments==2.6.1
      django-cors-headers==3.4.0

### Step 4: Inside 'backend' folder create docker-compose.yml and add:
      ```dockerfile
      version: '3'
      services:
        db_postgres:
          image: postgres
          hostname: postgres
          environment:
            - POSTGRES_DB=postgres
            - POSTGRES_USER=postgres
            - POSTGRES_PASSWORD=postgres
          ports:
            - "5432:5432"
        web_django:
          build: .
          command: >
            sh -c "python manage.py migrate &&
                   python manage.py makemigrations &&
                   python manage.py runserver 0.0.0.0:8000"
          volumes:
            - .:/code
          ports:
          - "8000:8000"
          depends_on:
            - db_postgres
          environment:
            WAIT_HOSTS: postgres:5432,
       '''   
### Step 5: Inside 'backend' folder create your django project:

    $ sudo docker-compose run web django-admin startproject djangodocker .
    
### Step 6: In your project directory, edit the djangodocker/setting.py:
    ```python
    # setting.py 
    
    # Replace DATABASES with: 
    
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.postgresql',
            'NAME': 'postgres',
            'USER': 'postgres',
            'PASSWORD': 'postgres',
            'HOST': 'db_postgres',
            'PORT': 5432,
        }
    }
    
    # Add 'rest_framework' and 'corsheaders' to your INSTALLED_APPS
    
    INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework', #used to create de api
    'corsheaders', #used for exporting the api to react
    ]
      
    # Edit ALLOWED_HOSTS
    
    ALLOWED_HOSTS = ['*']
    '''
    
 ### Step 7: Build the container (make sure you're in the right directory)
  
    $ docker build -t django-backend .
    
### Step 8: Run your app (make sure you're in the right directory)

    $ docker-compose up
    
At this point, your Django app should be running at port 8000 on your Docker host. On Docker Desktop for Mac and Docker Desktop for Windows, go to http://localhost:8000 on a web browser to see the Django welcome page.

!!! Make sure you add a secret key in settings.py before anything. You can generate one using https://djecrety.ir/ !!!

### Step 9: Create the react app inside 'frontend' folder

   ### a) Install react globally
      
      $ npm install -g create-react-app@3.4.1
      
   ### b) Generate new app
   
      $ npm init react-app reactdocker --use-npm
      $ cd reactdocker
      
   ### c) Create a Dockerfile (make sure you're in 'reactdocker' folder)
      ```dockerfile
      FROM node:13.12.0-alpine

      WORKDIR /app

      ENV PATH /app/node_modules/ .bin:$PATH

      COPY package.json ./
      COPY package-lock.json ./
      RUN npm install --silent
      RUN npm install react-scripts@3.4.1 -g --silent
      RUN npm install axios classnames --save
      COPY . ./

      CMD ["npm", "start"]
      '''
   ### d) Add a .dockerignore 
   
      node_modules
      build
      .dockerignore
      Dockerfile
      Dockerfile.prod
      
   ### e) Build the Docker image 
   
      $ docker build -t reactdocker:dev .
      
   ### f) Create a docker-compose.yml file and add
      ```dockerfile
      version: '3.7'

      services:
        sample:
          container_name: sample
          build:
            context: .
            dockerfile: Dockerfile
          volumes:
            - '.:/app'
            - '/app/node_modules'
          ports:
            - 3001:3000
          environment:
            - CHOKIDAR_USEPOLLING=true
          stdin_open: true
      '''
   ### g) Build the container
   
      $ docker-compose up -d --build
      
  ### h) Run the container
  
      $ docker-compose up 
      
Open your browser to http://localhost:3001/ and you should see the app. Try making a change to the App component within your code editor. You should see the app hot-reload. Kill the server once done.
      
That's all! I know it was a long journey but it pays off :+1: 

All this information can be found on:
- https://docs.docker.com/compose/django/
- https://mmorejon.io/en/blog/start-django-project-with-docker/
- https://mherman.org/blog/dockerizing-a-react-app/
