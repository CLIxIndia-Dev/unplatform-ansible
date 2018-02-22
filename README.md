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

Note that if your downloaded ePubs came with the old-style "Grades" format (i.e. the directory contains a `Grade 9` folder), you'll have to remove that Grades directory. For example:

```
English/
  |- Standard English/
    |- Grade 9/
      |- Unit 0/
```

Should become:

```
English/
  |- Standard English/
    |- Unit 0/
```

You will also have to make sure that all the URLs point to the deployed server, if not `localhost`. You have to do this in several places:

* `unplatform`
  * In the OEA config, `scripts/oea_build_config/_head.html`, all the host and service URLs.
  * In the content player,  `scripts/content_player_build_config/application.html`, the `loggingApiUrl`.
  * In `settings.py`, the `qbank` logging URL.
* `qbank`
  * In `dlkit_configs/configs.py`, set `ABS_PATH` and `TEST_ABS_PATH` to ''.
  * In `dlkit_configs/configs.py`, set `DATA_STORE_PATH` and `STUDENT_RESPONSE_DATA_STORE_PATH` to full paths to the files.
  * In `dlkit_configs/configs.py`, `FILESYSTEM_ADAPTER_1` change `urlHostname`.
  * Run a MongoDB script to convert all the `AssetContent` `url`s to full paths, instead of the relative paths used on the authoring server.
* ePubs
  * Run a shell command to convert all the embedded assessment URLs from `https://localhost:8888` to the desired hostname. For example, from the `modules` directory: `grep -lZ -r -e 'https://localhost:8888' . | xargs -0 -n1 sed -i.bak 's|https://localhost:8888|https://www.example.com|'`
