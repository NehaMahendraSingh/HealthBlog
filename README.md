# HealthBlog
**Prerequisites
1) AstraDB - Create a new Database 
2) Download django-cassandra-engine from https://pypi.org/project/django-cassandra-engine/        
Pip3 install django-cassandra-engine
#Allows us to integrate Cassandra with Django

**Building a Django Project
1) Start a new Django project “healthblog”- django-admin startproject healthblog<br />
2) Create a new app called “blog” from the root directory of the project - django-admin startapp blog.<br />

**Connect Database with our Django project**
1)Connect using the Python driver. 
2)Follow steps 1,2 in from Astra Connect I.e Download the Cassandra-driver for integration of AstraDB with python - pip3 install cassandra-driver.
3)Download the secure connect bundle provided by AstraDB and move it to the root directory of our project.
4)Create a Database administrator token
	Organization Settings -> Select role -> “database administrator” .
5) Download the token and move it to the root directory.


**Open this project in any IDE of your choice**
#healthblog>Settings.py
We need to import :
from cassandra import ConsistencyLevel
from cassandra.cluster import Cluster
from cassandra.auth import PlainTextAuthProvider


In the INSTALLED_APPS section - 
1)Add our app - ‘blog’
2)Add cassandra engine - ‘django-cassandra-engine’

**Let’s configure the database**
‘ENGINE’ : ‘django-cassandra-engine’
‘NAME’ : <key-space>
#connect using client ID and secret from the token we saved in the root directory


'OPTIONS': {
            'connection': {
                'auth_provider': PlainTextAuthProvider(('CLIENT ID', 'CLIENT SECRET’),
#Cloud - connect to the secure bundle we saved in the root directory
                'cloud': {
                    'secure_connect_bundle': 'secure-connect-healthblog.zip'
                }
            }
        }

Create a Django model in our app and sync it with our database
#blog>models.py


from django.db import models
#python Library
import uuid 
from datetime import datetime
from cassandra.cqlengine import columns
from django_cassandra_engine.models import DjangoCassandraModel

# Create your models here.
class PostModel(DjangoCassandraModel):
    id = columns.UUID(primary_key=True, default=uuid.uuid4) 
    title = columns.Text(required=True)
    body = columns.Text(required=True)
    created_at = columns.DateTime(default=datetime.now)

Let's sync our model with AstraDB and check if all the connections are set right.

On your terminal from the project’s root directory : python3 manage.py syncdb

''''
    Expected Output - Creating keyspace health_blog [CONNECTION default] ..
Syncing blog.models.PostModel
    ''''

Now we can try by running a query from Astra CQL console:
use healthblog;
SELECT * FROM PostModel;

