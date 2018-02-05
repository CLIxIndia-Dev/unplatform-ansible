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
