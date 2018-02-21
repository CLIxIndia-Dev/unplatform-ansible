Ansible playbooks for deploying Unplatform2 + QBank, on Ubuntu.
Tested on Ubuntu 16.0.4 with Ansible 2.4.

On a clean install:

```
$ sudo apt install python-pip git
$ pip install virtualenv virtualenvwrapper

Edit your .bashrc or similar file (depending on shell) to include:
  export WORKON_HOME=$HOME/.virtualenvs
  export PROJECT_HOME=$HOME/Devel
  source /usr/local/bin/virtualenvwrapper.sh OR source ~/.local/bin/virtualenvwrapper.sh (Ubuntu 16.04)

$ source ~/.bashrc
$ mkvirtualenv ansible
$ git clone https://github.com/CLIxIndia-Dev/unplatform-ansible.git
$ cd unplatform-ansible
$ pip install -r requirements.txt
$ ansible-playbook -i hosts main.yml --ask-become-pass --ask-vault-pass
```

Now you'll have to initialize the MongoDB replication set in the Mongo shell.
ONLY do this the first time you set up the server. The replication set
**only** needs to be initialized once.

```
$ mongo
> rs.initiate({_id: "dlkit", version: 1, members: [{_id: 0, host: "localhost:27017"}]})
> exit
```

Now you'll also have to download a set of valid data (ePubs and matching
assessment data, in MongoDB format). Only do this whenever you want to load
new data. Should be rare.

```
$ unzip modules.zip
$ unzip prod-data-dump.zip
$ # we want to keep our current Tools iframes, so delete the downloaded ones
$ rm -rf modules/Tools
$ mv modules/* /var/www/html/unplatform/modules/
$ mv prod-data-dump/CLIx/datastore/* /var/www/html/qbank/webapps/CLIx/datastore/
$ mongorestore --drop prod-data-dump/CLIx/mongodump
```
