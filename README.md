# HealthBlog
## Prerequisites<br />
* AstraDB - Create a new Database 
* Download `django-cassandra-engine` from (https://pypi.org/project/django-cassandra-engine/)   
```
Pip3 install django-cassandra-engine
```

## Building a Django Project<br />
* Start a new Django project 
```
django-admin startproject <project_name>
```
* Create a new app from the root directory of the project 
```
django-admin startapp <app_name>
```

## Connect Database with Django project<br />
* Connect using the `Python driver`. <br />
* Follow steps 1,2 in from `Astra Connect` I.e Download the `Cassandra-driver` for integration of AstraDB with python 
```
pip3 install cassandra-driver
```
* Download the `secure connect bundle` provided by AstraDB and move it to the root directory of our project.<br />
* Create a `Database administrator token`. <br />
>Organization Settings -> Select role -> `database administrator`.<br />
* Download the token and move it to the root directory.<br />


**Open this project in any IDE of your choice** 
<br />

#healthblog>Settings.py<br />

* Import the following :<br />

```
from cassandra import ConsistencyLevel
from cassandra.cluster import Cluster
from cassandra.auth import PlainTextAuthProvider
```



* In the INSTALLED_APPS section - <br />
1. Add our app - ‘blog’<br />
2. Add cassandra engine - ‘django-cassandra-engine’<br />

## Configure the database<br />

```
‘ENGINE’ : ‘django-cassandra-engine'
‘NAME’ : ('key-space') from the database
```

#connect using client ID and secret from the token we saved in the root directory

```
'OPTIONS': 
            'connection':	    
                'auth_provider': PlainTextAuthProvider(('CLIENT ID', 'CLIENT SECRET’),
				
#Cloud - connect to the secure bundle we saved in the root directory

                'cloud': {
                    'secure_connect_bundle': 'secure-connect-healthblog.zip
                }
            }   
        }
	
```

* Create a Django model in our app and sync it with our database<br />
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

 ```
 python3 manage.py syncdb
 ```
 ***Expected Output - 
 <br />
 Creating keyspace health_blog [CONNECTION default] ..
	<br />
    Syncing blog.models.PostModel***


<br />

* Now we can try by running a query from Astra CQL console: <br />

```
use healthblog;
SELECT * FROM PostModel;
```

* Test out by creating a new object

#Open Shell and run the following commands

```
Python3 manage.py shell
>>>From blog.models import PostModel’
>>> p = PostModel.objects.create (title = ‘ ‘, body = ‘’)
>>>p.save()
```

* Rerun the query on CQL to verify

## Frontend

* Set-up the template and static files to render front-end <br />
  Please copy it from the repo <br />



* Set-up the base directories for templates and static files <br />
#healthblog > Settings.py > templates
```
‘DIRS’ : [os.path.join(BASE_DIR, ‘templates’)]
```
#healthblog > Settings.py > STATIC_URL

```
STATIC_ROOT = os.path.join(BASE_DIR, ‘staticfiles’),
STATICFILES_DIRS = (os.path.join(BASE_DIR, ‘static’)
```

* Create a new file in the app: <urls.py> to specify the URLs and configure the views accordingly<br />

```
from django.urls import path
from . import views

urlpatterns = [
    path('', views.home, name=‘home'),  
    path('newpost', views.newpost, name='newpost'),
    path('posts/<str:pk>', views.posts, name='posts'),
]
```

#blog > views.py

```
from django.shortcuts import render, redirect
from .models import PostModel

# Create your views here.
def home(request):
    posts_list = sorted(list(PostModel.objects.all()), key=lambda p: p.created_at)
    return render(request, 'index.html', {'posts': posts_list})

def posts(request, pk):
    post = PostModel.objects.get(id=pk)
    return render(request, 'post.html', {'post':post})

def newpost(request):
    if request.method == 'POST':
        title = request.POST['title']
        body = request.POST['body']

        new_post = PostModel.objects.create(title=title, body=body)
        new_post.save
        return redirect('/')
    else:
        return render(request, ‘newpost.html')
```

#healthblog > urls.py 
```
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('blog.urls'))
]
```

* To run the project execute the command:
```
Python3 manage.py rumserver
```







