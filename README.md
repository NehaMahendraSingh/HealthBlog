# HealthBlog
## Prerequisites<br />
* AstraDB - Create a new Database 
* Download 'django-cassandra-engine' from (https://pypi.org/project/django-cassandra-engine/)   
>Pip3 install django-cassandra-engine
#Allows us to integrate Cassandra with Django

## Building a Django Project<br />
* Start a new Django project “healthblog”- django-admin startproject healthblog<br />
* Create a new app called “blog” from the root directory of the project - django-admin startapp blog.<br />

## Connect Database with our Django project<br />
* Connect using the 'Python driver'. <br />
* Follow steps 1,2 in from 'Astra Connec't I.e Download the Cassandra-driver for integration of AstraDB with python > (pip3 install cassandra-driver)<br />
* Download the secure connect bundle provided by AstraDB and move it to the root directory of our project.<br />
* Create a 'Database administrator token'<br />
>Organization Settings -> Select role -> “database administrator” .<br />
* Download the token and move it to the root directory.<br />


###### Open this project in any IDE of your choice<br />


#healthblog>Settings.py<br />


We need to import :<br />


from cassandra import ConsistencyLevel<br />

from cassandra.cluster import Cluster<br />

from cassandra.auth import PlainTextAuthProvider<br />



In the INSTALLED_APPS section - <br />
* Add our app - ‘blog’<br />
* Add cassandra engine - ‘django-cassandra-engine’<br />

## Configure the database<br />

‘ENGINE’ : ‘django-cassandra-engine'

‘NAME’ : ('key-space') from the database

#connect using client ID and secret from the token we saved in the root directory<br />

'OPTIONS': 

            'connection':
	    
                'auth_provider': PlainTextAuthProvider(('CLIENT ID', 'CLIENT SECRET’),
		
#Cloud - connect to the secure bundle we saved in the root directory

                'cloud': {
		
                    'secure_connect_bundle': 'secure-connect-healthblog.zip
		    
                }
		
            }
	    
        }
	

Create a Django model in our app and sync it with our database<br />
#blog>models.py

```

from django.db import models

import uuid 

from datetime import datetime

from cassandra.cqlengine import columns

from django_cassandra_engine.models import DjangoCassandraModel

```


###### Create your models here.<br />
```


class PostModel(DjangoCassandraModel):<br />
    id = columns.UUID(primary_key=True, default=uuid.uuid4) <br />
    title = columns.Text(required=True)<br />
    body = columns.Text(required=True)<br />
    created_at = columns.DateTime(default=datetime.now)<br />
    
```
    

* Let's sync our model with AstraDB and check if all the connections are set right.<br />

* On your terminal from the project’s root directory : 

> python3 manage.py syncdb<br />

````
```
    Expected Output - Creating keyspace health_blog [CONNECTION default] ..
Syncing blog.models.PostModel
```
````
<br />

* Now we can try by running a query from Astra CQL console: <br />


use healthblog;

SELECT * FROM PostModel;



