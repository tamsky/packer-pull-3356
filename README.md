# packer-pull-3356
Simple example to show use of new flag

Directory layout:
├── ansible.cfg
├── inventories/
│   └── elasticsearch
├── playbooks/
│   ├── elasticsearch.yml
│   └── full-stack.yml
├── roles/
│   └── elasticsearch/
│       └── tasks/
│           └── main.yml
└── util/
    └── packer/
            └── packer.yml

Assume packer.yml is being run using `cd util/packer/ ; packer build packer.yml`.



Without the new flag `playbook_file_dir` when files are copied to
the remote host, with `playbook
lit is not possible to use playbooks that `include:` other playbooks.
wind up not in the same directory

## working command line run of ansible (cwd == root directory of this repo):

    # ansible-playbook -i inventories/elasticsearch playbooks/full-stack.yml
    
    PLAY [elasticsearch] **********************************************************
    
    GATHERING FACTS ***************************************************************
    ok: [127.0.0.1]
    
    TASK: [elasticsearch | we made it] ********************************************
    ok: [127.0.0.1] => {
        "msg": "This is the task you were hoping for."
        }
    
    PLAY RECAP ********************************************************************
    127.0.0.1                  : ok=2    changed=0    unreachable=0    failed=0


## non-working packer build

    # packer validate
    packer build packer.json


### non-working Output

    root@vagrant-ubuntu-trusty-64:/vagrant/util/packer# ./packer build packer.json
    docker output will be in this color.
    
    ==> docker: Creating a temporary directory for sharing data...
    ==> docker: Pulling Docker image: ansible/ubuntu14.04-ansible:stable
        docker: stable: Pulling from ansible/ubuntu14.04-ansible
        docker: 0105f98ced6d: Already exists
        docker: 66395c31eb82: Already exists
        docker: 002fa881df8a: Already exists
        docker: a005e6b7dd01: Already exists
        docker: a678d4d95aef: Already exists
        docker: 7102acfbffc6: Already exists
        docker: 5007f52aeb28: Already exists
        docker: Digest: sha256:89bd35c622c295b81c58f365abb36c88703ab1c08266603f8c37f85b48cd9d28
        docker: Status: Image is up to date for ansible/ubuntu14.04-ansible:stable
    ==> docker: Starting docker container...
        docker: Run command: docker run -v /root/.packer.d/tmp/packer-docker771336864:/packer-files -d -i -t ansible/ubuntu14.04-ansible:stable /bin/bash
        docker: Container ID: 50398c1cc2d26cb0083b6e6420da423df6ca0b43f8d320839ae7c37400eb7484
    ==> docker: Provisioning with Ansible...
        docker: Uploading Playbook directory to Ansible staging directory...
        docker: Creating directory: /tmp/packer-provisioner-ansible-local
        docker: Uploading main Playbook file...
        docker: Uploading inventory file...
        docker: Uploading group_vars directory...
        docker: Creating directory: /tmp/packer-provisioner-ansible-local/group_vars
        docker: Executing Ansible: cd /tmp/packer-provisioner-ansible-local && ANSIBLE_FORCE_COLOR=1 PYTHONUNBUFFERED=1 ansible-playbook /tmp/packer-provisioner-ansible-local/full-stack.yml -c local -i /tmp/packer-provisioner-ansible-local/packer-provisioner-ansible-local566957404
        docker: ERROR: file could not read: /tmp/packer-provisioner-ansible-local/elasticsearch.yml
    ==> docker: Killing the container: 50398c1cc2d26cb0083b6e6420da423df6ca0b43f8d320839ae7c37400eb7484
    Build 'docker' errored: Error executing Ansible: Non-zero exit status: 1



## with PR#3356 applied, and correct playbook_file_dir param:

# packer-with-3356 build packer-with-3356.json
