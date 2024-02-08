# Demo environment for Ansible

## Quickstart

After checkout:

- Run `generate-keys.sh` script. This will generate SSH keys for the demo environment.
- Run `docker-compose build`. This will build the necessary Docker images.
- Run `docker-compose up`. This wil start the demo environment.
- Run `docker exec -i -t main /bin/bash` to enter the main container.

## Description

## Naming and labeling convention

All docker objects created by this demo have a certain set of labels attached. The labels `CUSTOMER` and `PROJECT` have the same value for all objects:

- `CUSTOMER=com.gft.devops-event`
- `PROJECT=com.gft.devops-event.ansible-demo-1`

The values of the labels `MODULE`, `COMPONENT` and `NAME` differ from object to object.
All objects created by this demo can be found by adding the following label selector to the `docker` command:
```
    --filter "label=CUSTOMER=com.gft.devops-event" \
    --filter "label=PROJECT=com.gft.devops-event.ansible-demo-1"
```

All docker objects follow this naming convention:
```
    <PROJECT>.<MODULE>.<COMPONENT>.<NAME>
```
Example: `com.gft.devops-event.ansible-demo-1.infrastructure.generate-keys.main`

Exception: The actual containers, which are named for ease of use and better comprehensability of the demo.

## Cleanup

The `cleanup.sh` script will remove all containers, images, and volumes created by this demo. It will use the above mentioned label filter to find the objects to remove.

## Container images

### Image for key generation

The image `com.gft.devops-event.ansible-demo-1.infrastructure.generate-keys.main:1.0.0` will create all necessary SSH keys for the demo environment. The script `generate-keys.sh` builds this image and copies the keys into the correct place.

### Demo images

The image `com.gft.devops-event.ansible-demo-1.infrastructure.ansible-host:1.0.0` is meant for interactive use. It contains ansible and the demo files.

The images `com.gft.devops-event.ansible-demo-1.app.worker.1:1.0.0`, `com.gft.devops-event.ansible-demo-1.app.worker.2:1.0.0` and `com.gft.devops-event.ansible-demo-1.app.worker.3:1.0.0` are meant to be used as targets for the ansible playbooks. They contain a very basic linux distribution with python3 installed. They all run an ssh daemon on port 22, allowing the ansible host to connect to them.

The image `com.gft.devops-event.ansible-demo-1.app.database.main:1.0.0` is meant to be used as a database server. It contains a very basic linux distribution with python3 installed. It runs an ssh daemon on port 22 and a postgresql server on port 5432.

## Containers

### Container `main`

This container is meant to be the interactive demo environment. Enter it with `docker exec -i -t main /bin/bash`. You'll be the user `ansible` and you'll find yourself in your homedirectory. Everything is configured to connect to the other containers with ssh.

### Container `worker_1` and `worker_2`

These container contain two different linux distributions and run an ssh daemon on port 22. They are meant to be used as targets for the ansible playbooks. `worker_1` runs a debian distribution, `worker_2` runs an alpine distribution.

### Container `database`

This container runs a postgresql database `main`. It allows admin access to the database with the user `postgres` and the password `admin`. The database is reachable on port 5432. Like the other containers it too runs an ssh daemon on port 22.

### Container `comgftdevops-eventansible-demo-1-worker_3-1` and `comgftdevops-eventansible-demo-1-worker_3-2`

These are deployed as replicas by docker-compose. They are based on the same container image as `worker_2` and are meant to be used to demonstrate the dynamic inventory feature of ansible.

# Playbooks

In the container main you'll find the directories `/home/ansible/step-1`, ... These directories contain the individual steps of the demonstration.

Except for the steps showing the dynamic inventory feature, all steps contain an inventory.yaml file, that contains the containers `worker_1`, `worker_2` and `database` as targets. The individual steps may add or remove information from this file, according to the step's purpose.

## `step-1`

This shows the basic usage of ansible. As a first step, you should call `ansible-playbook -i inventory.yaml load-ssh-hostkeys.yaml`. This will ensure that the ssh host-keys of the target containers are known to the ansible host. After that, you can call `ansible-playbook -i inventory.yaml ping.yaml` to check if the target containers are reachable.
