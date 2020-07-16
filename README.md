# Django-React-Docker

### Step 1: Create an empty project directory and inside it, 2 more folders: backend and frontend.

### Step 2: Inside 'backend' folder create Dockerfile

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
      
### Step 3: Inside 'backend' folder create requirements.txt and add:

      Django==2.2.14
      psycopg2-binary>=2.7.4
      djangorestframework==3.11.0
      Pygments==2.6.1
      django-cors-headers==3.4.0
      
### Step 3: Inside 'backend' folder create docker-compose.yml and add:

      version: '3'
      services:
        db:
          image: postgres
          hostname: postgres
          environment:
            - POSTGRES_DB=postgres
            - POSTGRES_USER=postgres
            - POSTGRES_PASSWORD=postgres
          ports:
            - "5432:5432"
        web:
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
            - db
          environment:
            WAIT_HOSTS: postgres:5432,
            
### Step 4: Inside 'backend' folder create your django project:

    sudo docker-compose run web django-admin startproject djangodocker .
    
### Step 5: In your project directory, edit the djangodocker/setting.py:

    # setting.py -> Replace DATABASES with: 
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.postgresql',
            'NAME': 'postgres',
            'USER': 'postgres',
            'PASSWORD': 'postgres',
            'HOST': 'db',
            'PORT': 5432,
        }
    }
  
 ### Step 6: Build the container(make sure you're in the right directory)
  
    docker build -t django-backend .
    
### Step 7: Run your app(make sure you're in the right directory)

    docker-compose up
    
At this point, your Django app should be running at port 8000 on your Docker host. On Docker Desktop for Mac and Docker Desktop for Windows, go to http://localhost:8000 on a web browser to see the Django welcome page.

