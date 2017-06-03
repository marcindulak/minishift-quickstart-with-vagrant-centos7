# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_DEFAULT_PROVIDER'] = 'libvirt'

Vagrant.configure(2) do |config|
  config.vm.define 'minishift' do |machine|
    machine.vm.box = 'centos/7'
    machine.vm.box_url = machine.vm.box
    machine.vm.provider 'libvirt' do |p|
      p.memory = 3072
      p.cpus = 2
      p.nested = true
    end
  end
  # disable IPv6 on Linux
  $linux_disable_ipv6 = <<SCRIPT
sysctl -w net.ipv6.conf.default.disable_ipv6=1
sysctl -w net.ipv6.conf.all.disable_ipv6=1
sysctl -w net.ipv6.conf.lo.disable_ipv6=1
SCRIPT
  # setenforce 0
  $setenforce_0 = <<SCRIPT
if test `getenforce` = 'Enforcing'; then setenforce 0; fi
#sed -Ei 's/^SELINUX=.*/SELINUX=Permissive/' /etc/selinux/config
SCRIPT
  # stop firewalld
  $systemctl_stop_firewalld = <<SCRIPT
systemctl stop firewalld.service
SCRIPT
  config.vm.define "minishift" do |machine|
    # check for nested virtualization!
    machine.vm.provision :shell, :inline => 'egrep -c "(vmx|svm)" /proc/cpuinfo > /dev/null'
    machine.vm.provision :shell, :inline => 'hostname minishift', run: 'always'
    machine.vm.provision :shell, :inline => $setenforce_0, run: 'always'
    machine.vm.provision :shell, :inline => $linux_disable_ipv6, run: 'always'
    machine.vm.provision :shell, :inline => 'yum -y install wget git'
    machine.vm.provision :shell, :inline => 'yum -y groups install "X Window System"'
    machine.vm.provision :shell, :inline => 'systemctl set-default graphical.target'
  end
end
