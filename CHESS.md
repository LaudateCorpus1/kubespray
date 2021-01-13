# Kubespray deployment on chess.com hardware

## Prerequisites

You need python and supporting packages in order to run ansible. The simplest
is to use virtualenvwrapper to create a separate python installation for
kubespray packages. That way it will not interfere with system packages. This
is an optional step for GNU/Linux, but **mandatory** for MacOS.

Also make sure devtools is setup in ~/.ssh/config for port 2022, username, hostname, etc.
Follow [ssh directions](https://app.tettra.co/teams/chesscom/pages/ssh-config).

```bash
yum|rpm|apt install virtualenvwrapper
# or brew install virtualenvwrapper
```

Then you can start and create a virtualenv. If you're going to run kubespray often it is a good idea to put the source and export lines below in your shell profile (.zprofile,.bashprofile,.profile,etc):

```bash
# Initialize virtualenv
source $(which virtualenvwrapper.sh)
export WORKON_HOME=/path/to/virtualenvs # (this is up to you, ~/.virtualenvs is a common choice).

# Create the virtualenv
mkvirtualenv chesscom-kubespray

# Now, whenever you want to run ansible commands, make sure you
# execute workon to activate the virtualenv
# (which will put ansible and other components in $PATH)
workon chesscom-kubespray

# Now we're ready for kubespray
git clone git@github.com:chesscom/kubespray chesscom-kubespray
cd chesscom-kubespray
git checkout -b chesscom-dev

pip install -r requirements.txt
```

`cd` into `inventory/chesscom` and take a look at the following files to familiarize yourself:

  - inventory.ini
  - group_vars/all/all.yml
  - group_vars/k8s-cluster/k8s-cluster.yml

## Deploy the cluster

When you're ready, you can deploy the cluster. Ansible will ssh through the bastion host (devtools) and then ssh into the cluster hosts to install software, configure docker and many other services, and run containers. This took about 57mins the first time on fresh hosts:

```bash
ansible-playbook -v -i inventory/chesscom/inventory.ini -b -u <username> -e 'bastion_user=<username>' cluster.yml
```

###### Note: the username for bastion_user can differ from the username in `-u <username>`.

Cluster configuration will be in /etc/kubernetes on the master nodes. You can interact with the cluster:

```bash
kubectl --kubeconfig=/etc/kubernetes/admin.conf -n kube-system get nodes
```
