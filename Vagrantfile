# -*- mode: ruby -*-
# vi: set ft=ruby :

require_relative './util/vagrant/key_authorization'

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ubuntu/trusty64"

  # vagrant-cachier boilerplate:
  if Vagrant.has_plugin?("vagrant-cachier")
    # Configure cached packages to be shared between instances of the same base box.
    # More info on http://fgrehm.viewdocs.io/vagrant-cachier/usage
    config.cache.scope = :box

    # OPTIONAL: If you are using VirtualBox, you might want to use that to enable
    # NFS for shared folders. This is also very useful for vagrant-libvirt if you
    # want bi-directional sync
    ## config.cache.synced_folder_opts = {
    ##   type: :nfs,
    ##   # The nolock option can be useful for an NFSv3 client that wants to avoid the
    ##   # NLM sideband protocol. Without this option, apt-get might hang if it tries
    ##   # to lock files needed for /var/cache/* operations. All of this can be avoided
    ##   # by using NFSv4 everywhere. Please note that the tcp option is not the default.
    ##   mount_options: ['rw', 'vers=3', 'tcp', 'nolock']
    ## }
    # For more information please check http://docs.vagrantup.com/v2/synced-folders/basic_usage.html
  end
  

  # copy the local user's key into root's authorized_keys:
  authorize_key_for_root config, '~/.ssh/id_dsa.pub', '~/.ssh/id_rsa.pub'

# only if using NFS
#  config.vm.network "private_network", ip: "192.168.33.10"
  config.vm.network "forwarded_port", guest: 80, host: 8080

  # Because it is so different from how we employ ansible in production...
  # We are going to try to avoid this type of use of the ansible vagrant provisioner:
  #
  #  config.vm.provision :ansible do |ansible|
  #    ansible.playbook = "playbook.yml"
  #  end
end
