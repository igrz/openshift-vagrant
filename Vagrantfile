# -*- mode: ruby -*-
# vi: set ft=ruby :
#
# Copyright 2017 Liu Hongyu
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

OPENSHIFT_RELEASE = "3.11"
OPENSHIFT_ANSIBLE_BRANCH = "release-#{OPENSHIFT_RELEASE}"
NETWORK_BASE = "172.18.15"
INTEGRATION_START_SEGMENT = 101

Vagrant.configure("2") do |config|

  config.vm.box = "centos/7"
  config.vm.box_check_update = false

  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.ignore_private_ip = false

  config.vm.provision "shell", inline: <<-SHELL
    /vagrant/all.sh #{OPENSHIFT_RELEASE}
  SHELL

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "10240"
    vb.cpus   = "8"
  end

  # Define nodes
  (1..1).each do |i|
    config.vm.define "node0#{i}" do |node|
      node.vm.network "private_network", ip: "#{NETWORK_BASE}.#{INTEGRATION_START_SEGMENT + i}"
      node.vm.hostname = "node0#{i}.example.com"
    end
  end

  # Define master
  config.vm.define "master", primary: true do |node|
    node.vm.network "private_network", ip: "#{NETWORK_BASE}.#{INTEGRATION_START_SEGMENT}"

    node.hostmanager.aliases = %w(master.example.com etcd.example.com nfs.example.com console.openshift.example.com online-test.example.com mgmt-test.example.com cms-test.example.com sb-mt-test.example.com)

    #
    # Memory of the master node must be allocated at least 2GB in order to
    # prevent kubernetes crashed-down due to 'out of memory' and you'll end
    # up with
    # "Unable to restart service origin-master: Job for origin-master.service
    #  failed because a timeout was exceeded. See "systemctl status
    #  origin-master.service" and "journalctl -xe" for details."
    #
    # See https://github.com/kubernetes/kubernetes/issues/13382#issuecomment-154891888
    # for mor details.
    #
    node.vm.provider "virtualbox" do |vb|
      vb.memory = "4096"
      vb.cpus   = "4"
    end

    node.vm.provision "shell", inline: <<-SHELL
      # Setting hostname manually to prevent "Could not find csr for nodes" error
      # Issue: https://github.com/eliu/openshift-vagrant/issues/12
      # Workaround: https://bugzilla.redhat.com/show_bug.cgi?id=1625911#c43
      hostnamectl set-hostname master.example.com

      /vagrant/master.sh #{OPENSHIFT_RELEASE} #{OPENSHIFT_ANSIBLE_BRANCH} #{NETWORK_BASE}
    SHELL

    # Deploy private keys of each node to master
    if File.exist?(".vagrant/machines/master/virtualbox/private_key")
      node.vm.provision "master-key", type: "file", run: "never", source: ".vagrant/machines/master/virtualbox/private_key", destination: "/home/vagrant/.ssh/master.key"
    end

    if File.exist?(".vagrant/machines/node01/virtualbox/private_key")
      node.vm.provision "node01-key", type: "file", run: "never", source: ".vagrant/machines/node01/virtualbox/private_key", destination: "/home/vagrant/.ssh/node01.key"
    end
  end
end
