Introduction
The basic development platform for small and medium-sized applications based on RBAC model permission control, the front and back ends are separated, the back-end adopts django+django-rest-framework, the front-end adopts vue+ElementUI, and the mobile terminal adopts uniapp+uView (can publish h5 and mini programs).

JWT certification, which can be used to implement auditing functions using simple_history, supports swagger

The built-in modules include Organization\User\Role\Position\Data Dictionary\File Vault\Scheduled Task\Workflow (most of the code has been uploaded, and the backend code is located in apps/wf).

Use the workflow suggestion database with Postgresql, the following preview environment because it is used sqlite so some JSON queries do not support, the usage method can refer to the loonflow documentation is basically the same, mainly simplified

Supports functional permissions (control to each interface) and simple data permissions (all, this level and below, peers and below, me, etc

Welcome to the issue

Partial screenshot
image image image

Preview the address
Preview the address directly used by runserver, account admin, password admin. Please exercise caution and do not change your password http://47.95.0.242:1111/

Boot (here are the steps to develop under Windows).
Django backend
Navigate to the server folder

Set up a virtual environment python -m venv venv

Activate the virtual environment .\venv\scripts\activate

Install dependent packages pip install -r requirements.txt

Modify the database connection server\settings_dev.py

Synchronize the database python manage.py migrate

Initial data can be imported python manage.py loaddata db.json Or use the SQLite database directly (the password of the hypermanagement account is admin, and the database will be reset every once in a while).

Create a super administrator python manage.py createsuperuser

Run the service python manage.py runserver 8000

Vue frontend
Navigate to the client folder

Install the node .js

Install dependent packages npm install --registry=https://registry.npm.taobao.org

Run the service npm run dev

nginx
Modify nginx.conf

listen 8012
location /media {
    proxy_pass http://localhost:8000;
}
location / {
    proxy_pass http://localhost:9528;
}
Run nginx .exe

Run
Open localhost:8012 to access

Interface documentation localhost:8000/docs

Background address localhost:8000/admin

Deployment
The settings_pro.py was used when it was deployed. Pay attention to modifications

It can be deployed separately from the front and back ends, and the nginx agent. It can also be packaged and placed in the server/vuedist folder, and then executed collectstatic

Docker-compose mode runs
Frontend ./client and backends ./server There are Dockerfiles in the directory, if you need to build the image separately, you can build it yourself.

Here we mainly talk about docker-compose starting this way.

Modify the docker-compose.yml file as commented. There are two main services inside, one is backendBackend, one is frontendFront.

The default is to run the backend and frontend in development mode. If you need to deploy a single machine and want to use docker-compose, the performance will be better when you change to production mode.

Start

cd <path-to-your-project>
docker-compose up -d
After successful startup, the access port is the same as the front, interface 8000 port, front-end port 8012, if you need to change, change docker-compose.yml

If you want to execute the command inside docker-compose exec < the service name > < command >

Take a chestnut:

If I want to execute a back-end generate data change command. python manage.py makemigrations

Then use the following statement

docker-compose exec backend python manage.py makemigrations
Philosophy
First of all, you must be able to use the django-rest-framework to understand the vue-element-admin front-end solution

This project adopts front-end routing, and the back-end reads the user permission code according to the user role and returns it to the frontend, which is loaded by the front-end (the core code is the perms attribute in the routing table and the checkpermission method).

The core code for back-end functional permissions overrides has_permission methods under server/apps/system/permission.py to define perms permission code in APIView and ViewSet

Because data permissions are related to specific businesses, several rules are simply defined and has_object_permission methods are rewritten; Use as needed

Due to the complexity of the actual situation, it is recommended to write the DRF permission_class according to different situations

About scheduled tasks
Implemented using celery as well as django_celery_beat packages

Redis needs to be installed and started on the default port, and Worker and Beat started

Enter the virtual environment and start the worker: celery -A server worker -l info -P eventlet, Linux systems do not need to add -P eventlet

Enter the virtual environment and start beat: celery -A server beat -l info

Follow-up
The workflow module can refer to the implementation of loonflow and check its documentation (same logic, thanks to loonflow) Most of the code has been uploaded, see swagger

Next steps
The processing of functional permissions and data permissions has a large optimization space, which can achieve a more reasonable division of permissions, but the current code changes are large, which is under consideration
