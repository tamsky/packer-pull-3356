# packer-pull-3356
Simple example to show use of new flag

packer is being run using `cd util/packer/ ; packer build packer.json`

Without the proposed flag `playbook_file_dir`, when the
`playbook_file` is copied to the remote host, it is copied to the
`playbook_dir`.   Packer uses `playbook_dir` as the path to the top of the
ansible tree that is copied to the remote host.  Result is that the runtime
command-line playbook and the rest of the playbooks are not placed in
the same directory as they are on the packer host.


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

# provisioner directory tree after failure (without patch):

    Tree of /tmp/packer-provisioner-ansible-local/:
    |-- LICENSE
    |-- README.md
    |-- Vagrantfile
    |-- ansible.cfg
    |-- full-stack.yml  # <<<<----- playbook directory location that existing code forces
    |-- group_vars
    |   `-- all
    |-- inventories
    |   `-- elasticsearch
    |-- packer-provisioner-ansible-local005490379
    |-- playbooks
    |   |-- elasticsearch.yml
    |   `-- full-stack.yml
    |-- roles
    |   `-- elasticsearch
    |       `-- tasks
    |           `-- main.yml
    `-- util
        |-- packer
        |   |-- packer-with-3356.json
        |   `-- packer.json
        `-- vagrant
            `-- key_authorization.rb
    


## with PR#3356 applied, and new flag `playbook_file_dir` in use:

    # packer-with-3356 build packer-with-3356.json

root@vagrant-ubuntu-trusty-64:/vagrant/util/packer# packer-bin-with-3356-patch build packer-with-3356.json

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
        docker: Run command: docker run -v /root/.packer.d/tmp/packer-docker408422136:/packer-files -d -i -t ansible/ubuntu14.04-ansible:stable /bin/bash
        docker: Container ID: 31d6dbdff1b738d6385de0218c8db73bd91231fc06024aa1109eeeafee08ef5a
    ==> docker: Provisioning with Ansible...
        docker: Uploading Playbook directory to Ansible staging directory...
        docker: Creating directory: /tmp/packer-provisioner-ansible-local
        docker: Uploading main Playbook file...
        docker: Uploading inventory file...
        docker: Uploading group_vars directory...
        docker: Creating directory: /tmp/packer-provisioner-ansible-local/group_vars
        docker: Executing Ansible: cd /tmp/packer-provisioner-ansible-local && ANSIBLE_FORCE_COLOR=1 PYTHONUNBUFFERED=1 ansible-playbook /tmp/packer-provisioner-ansible-local/playbooks/full-stack.yml -c local -i /tmp/packer-provisioner-ansible-local/packer-provisioner-ansible-local183602261
        docker:
        docker: PLAY [elasticsearch] **********************************************************
        docker:
        docker: GATHERING FACTS ***************************************************************
        docker: ok: [127.0.0.1]
        docker:
        docker: TASK: [elasticsearch | we made it] ********************************************
        docker: ok: [127.0.0.1] => {
        docker: "msg": "This is the task you were hoping for."
        docker: }
        docker:
        docker: PLAY RECAP ********************************************************************
        docker: 127.0.0.1                  : ok=2    changed=0    unreachable=0    failed=0
        docker:
    ==> docker: Committing the container
        docker: Image ID: 8c741ae341b82fc9d734caaffab413fad07cd028f014b2551799f7feab0fc1db
    ==> docker: Killing the container: 31d6dbdff1b738d6385de0218c8db73bd91231fc06024aa1109eeeafee08ef5a
    Build 'docker' finished.

## Working tree after provisioner run:

    root@57f6bac05ed6:/# tree /tmp/packer-provisioner-ansible-local/
    /tmp/packer-provisioner-ansible-local/
    |-- LICENSE
    |-- README.md
    |-- Vagrantfile
    |-- ansible.cfg
    |-- group_vars
    |   `-- all
    |-- inventories
    |   `-- elasticsearch
    |-- packer-provisioner-ansible-local183602261
    |-- playbooks
    |   |-- elasticsearch.yml
    |   `-- full-stack.yml
    |-- roles
    |   `-- elasticsearch
    |       `-- tasks
    |           `-- main.yml
    `-- util
        |-- packer
        |   |-- packer-with-3356.json
        |   `-- packer.json
        `-- vagrant
            `-- key_authorization.rb
