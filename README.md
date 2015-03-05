# Ansible Django Deployment playbook.

Please not that the MIT License is in effect only if the [license pull request](https://github.com/airtonix/ansible-django/pull/3) is accepted. 

This playbook works with (or will aim to work with) the following technologies:

* Django
* Git
* uWsgi
* Nginx
* PostgreSQL
* Celery
* Haystack
* Elastic Search
* Redis


## Todo

* Security
    * close all ports and open only those required
    * dbservers:
        * only allow access into postgres port from appservers ip addresses

* Database Node
    * celery
    * haystack
    * elasticsearch

* Monitoring
    * Nagios

* Auto Scaling
    * nagios triggers to automate ansible scaling


## Installation

* git clone this repo to a sensible location
* edit ~/.bash_rc (or other such file) to include an alias to the `play` script.
    `alias playbook='~/Dev/ansible/play'`
* copy the `inventory` directory to your django project source tree (see below) and put all sensitive information in this folder (assuming your project is private).
* copy the Vagrantfile to the same location.
* modify line 51 to point at this repo you cloned:
`ansible.playbook = "~/Dev/ansible/playbook/site.yml"`

## Minimum Django Applications

I make some assumptions about the packages that you'll use:

* django-postgrespool==0.2.4
* hiredis==0.1.1
* django-redis-cache==0.10.0


## Running deployments

just a small example of things you can do :

```
playbook deployment/inventory/production all
playbook deployment/inventory/production webservers
playbook deployment/inventory/production dbservers
playbook deployment/inventory/production webservers --tags=pip
```

## Testing deployments

* install virtualbox first
* edit inventory/vagrant to suit your desired scenerio
* get the machines created

```
$ pwd
/home/zenobius/Dev/websites/my-new-project/

$ cd deployment

$ ls -algh
total 4.0K
drwxrwxr-x 1 zenobius   56 Oct 23 08:57 .
drwxrwxr-x 1 zenobius   96 Oct 23 08:56 ..
drwxr-xr-x 1 zenobius   44 Oct 23 07:21 inventory
-rw-rw-rw- 1 zenobius 2.0K Oct 23 08:47 Vagrantfile

$ vagrant up
... snip hundreds of lines about creating virtualmachines ...

```
Then begin provisioning process

```
$ vagrant provision
... snip hundreds of lines about ansible playbook output ...

```

## Using the playbook

* firstly, make sure your project directory layout matches the layout described below
* ensure you've followed the steps above in Installation.
* in your deployment directory :

For the entire infrastructure
```
$ playbook inventory/production site
```

To setup the database servers
```
$ playbook inventory/production dbservers
```

To run the tasks dealing with only installed/updating packages
```
$ playbook inventory/production appservers --tags=packages
```




## Django Project Layout

In your django project, you should organise your settings module like so :

* git_root
    * deployment/
        * inventory
    * requirements/
        * base.list
        * live.list
        * test.list
        * local.list
    * application/
        * `__init__.py`
        * manage.py
        * base/
            * `__init__.py`
            * wsgi.py
            * settings/
                * modules/
                    * `__init__.py`
                * `__init__.py`
                * default.py
                * local.py
                * live.py


### project_root/manage.py

```
#!/usr/bin/env python
#
# file: project_root/manage.py
#
import os
import sys

if __name__ == "__main__":
    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "base.settings.local")
    from django.core.management import execute_from_command_line
    execute_from_command_line(sys.argv)
```

### project_root/base/wsgi.py
```
import os
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "base.settings.live")
from django.core.wsgi import get_wsgi_application
application = get_wsgi_application()

```

### project_root/base/settings/live.py

```
from .default import *
from .modules.db import *
from .modules.cache import *
from .modules.celery import *

DEBUG = False
TEMPLATE_DEBUG = DEBUG

...

```

### project_root/base/settings/local.py

```
from .default import *


DEBUG = True
TEMPLATE_DEBUG = DEBUG

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': 'db.sqlite3',
    }
}
...

```
