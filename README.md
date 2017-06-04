-----------
Description
-----------

`minishift`'s quickstart guide https://docs.openshift.org/latest/minishift/getting-started/quickstart.html
performed on CentOS 7 using libvirt/kvm nested virtualization under `vagrant`.
Example requested at https://github.com/minishift/minishift/issues/974

![nodejs-ex](https://raw.github.com/marcindulak/minishift-quickstart-with-vagrant-centos7/master/screenshots/nodejs-ex.png)

Tested on Ubuntu 16.04 host.


------------
Sample Usage
------------

Install `vagrant` https://www.vagrantup.com/downloads.html

Make sure nested virtualization is enabled on your host (e.g. your laptop):

        $ modinfo kvm_intel | grep -i nested

and install, configure libvirt:

        $ sudo apt-get -y install kvm libvirt-bin
        $ sudo adduser $USER libvirtd  # logout and login again

Install https://github.com/pradels/vagrant-libvirt plugin dependencies:

        $ sudo apt-get install -y ruby-libvirt
        $ sudo apt-get install -y libxslt-dev libxml2-dev libvirt-dev zlib1g-dev

Disable the default libvirt network:

        $ virsh net-autostart default --disable
        $ sudo service libvirt-bin restart

and delete the default/images storage pool::

        $ virsh pool-destroy default
        $ virsh pool-destroy images
        $ virsh pool-undefine images

**Note** libvirt may have problems if ip6tables are not running.
Make also sure that no other virtualization (VirtualBox, etc.)
is active on the host machine.

Start the VM guest on which `minishift` will be installed:

        $ git clone https://github.com/marcindulak/minishift-quickstart-with-vagrant-centos7.git
        $ cd minishift-quickstart-with-vagrant-centos7
        $ vagrant plugin install vagrant-libvirt
        $ vagrant up --no-parallel

Configure `minishift` on the guest VM:

- configure the RPM repository https://copr.fedorainfracloud.org/coprs/marcindulak/minishift/, and install `minishift`:

            $ vagrant ssh minishift -c "sudo su - -c 'yum -y install wget git'"
            $ vagrant ssh minishift -c "sudo su - -c 'wget -q https://copr.fedorainfracloud.org/coprs/marcindulak/minishift/repo/fedora-rawhide/marcindulak-minishift-fedora-rawhide.repo -P /etc/yum.repos.d'"
            $ vagrant ssh minishift -c "sudo su - -c 'yum -y install minishift'"

- install `docker` https://docs.docker.com/engine/installation/linux/centos:

            $ vagrant ssh minishift -c "sudo su - -c 'yum -y install yum-utils device-mapper-persistent-data lvm2'"
            $ vagrant ssh minishift -c "sudo su - -c 'yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo'"
            $ vagrant ssh minishift -c "sudo su - -c 'yum -y install docker-ce'"
            $ vagrant ssh minishift -c "sudo su - -c 'mkdir -p /etc/docker&& echo { \\\"storage-driver\\\": \\\"devicemapper\\\" } > /etc/docker/daemon.json'"
            $ vagrant ssh minishift -c "sudo su - -c 'systemctl start docker'"
            $ vagrant ssh minishift -c "sudo su - -c 'docker run hello-world'"
            $ vagrant ssh minishift -c "sudo su - -c 'systemctl enable docker'"

- install `docker-machine-driver-kvm`:

            $ vagrant ssh minishift -c "sudo su - -c 'curl -L https://github.com/dhiltgen/docker-machine-kvm/releases/download/v0.10.0/docker-machine-driver-kvm-centos7 > /usr/local/bin/docker-machine-driver-kvm&& chmod +x /usr/local/bin/docker-machine-driver-kvm'"

- configure `libvirtd` on the `minishift` VM, add user `vagrant` to the `libvirt` group:

            $ vagrant ssh minishift -c "sudo su - -c 'yum -y install libvirt qemu-kvm'"
            $ vagrant ssh minishift -c "sudo su - -c 'systemctl start libvirtd'"
            $ vagrant ssh minishift -c "sudo su - -c 'systemctl enable libvirtd'"
            $ vagrant ssh minishift -c "sudo su - -c 'usermod -aG libvirt vagrant'"

- install a web browser (will be used to access the application deployed with `minishift`) and reboot the VM into X:

            $ vagrant ssh minishift -c "sudo su - -c 'yum -y install konqueror xorg-x11-fonts-Type1 && reboot'"
            $ sleep 30
            $ vagrant ssh minishift -c "sudo su - -c 'systemctl is-active docker'"
            $ vagrant ssh minishift -c "sudo su - -c 'systemctl is-active libvirtd'"

The `minishift` quickstart https://docs.openshift.org/latest/minishift/getting-started/quickstart.html consists of:

- Starting `minishift`:

            $ vagrant ssh minishift -c "minishift start --cpus=2 --memory=2048 --vm-driver=kvm"
            $ vagrant ssh minishift -c "minishift oc-env"

- Deploying an application:

            $ vagrant ssh minishift -c 'eval $(minishift oc-env) && oc new-app https://github.com/openshift/nodejs-ex -l name=myapp'
            $ vagrant ssh minishift -c 'eval $(minishift oc-env) && oc logs -f bc/nodejs-ex'
            $ vagrant ssh minishift -c 'eval $(minishift oc-env) && oc expose svc/nodejs-ex'
            $ ! timeout 30 vagrant ssh minishift -c 'minishift openshift service nodejs-ex -n myproject' -- -X
            $ vagrant ssh minishift -c 'minishift stop'

When done, destroy the test machine with:

        $ vagrant destroy -f


------------
Dependencies
------------

https://github.com/pradels/vagrant-libvirt


-------
License
-------

BSD 2-clause


--------
Problems
--------

