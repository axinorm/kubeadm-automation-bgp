# Variables
# Image à utiliser
IMAGE_NAME = "ubuntu/focal64"
IMAGE_VERSION = "20220419.0.0"
# RAM
MEM = 2048
# Nombre de CPU
CPU = 2
# Nombre de noeuds
K8S_WORKER_NB = 1
# Prefix pour le réseau du cluster Kubernetes
K8S_NODE_NETWORK_BASE = "10.0.0"
# Prefix pour le réseau client
CLIENT_NETWORK_BASE = "172.16.0"
# Prefix pour le réseau BGP
BGP_NETWORK_BASE = "172.16.254"

Vagrant.configure("2") do |config|
  config.ssh.insert_key = true

  # Configuration de la RAM et du CPU
  config.vm.provider "virtualbox" do |v|
    v.gui = false
    v.memory = MEM
    v.cpus = CPU
  end

  # Configuration du Router
  config.vm.define "router" do |router|
    router.vm.box = IMAGE_NAME
    router.vm.box_version = IMAGE_VERSION
    router.vm.synced_folder '.', '/vagrant', disabled: true

    router.vm.network "private_network", ip: "#{K8S_NODE_NETWORK_BASE}.1",
      netmask: '255.255.255.0', virtualbox__intnet: 'k8s-network'
    router.vm.network "private_network", ip: "#{CLIENT_NETWORK_BASE}.1",
      netmask: '255.255.255.0', virtualbox__intnet: 'client-network'
    router.vm.hostname = "router"
  end

  # Configuration du Master
  config.vm.define "k8s-master" do |k8s_master|
    k8s_master.vm.box = IMAGE_NAME
    k8s_master.vm.box_version = IMAGE_VERSION
    k8s_master.vm.synced_folder '.', '/vagrant', disabled: true

    k8s_master.vm.network "private_network", ip: "#{K8S_NODE_NETWORK_BASE}.10",
      netmask: '255.255.255.0', virtualbox__intnet: 'k8s-network'
    k8s_master.vm.hostname = "k8s-master"
  end

  # Configuration du/des Worker(s)
  (1..K8S_WORKER_NB).each do |i|
    config.vm.define "k8s-worker#{i}" do |k8s_worker|
      k8s_worker.vm.box = IMAGE_NAME
      k8s_worker.vm.box_version = IMAGE_VERSION
      k8s_worker.vm.synced_folder '.', '/vagrant', disabled: true

      k8s_worker.vm.network "private_network", ip: "#{K8S_NODE_NETWORK_BASE}.#{i + 50}",
        netmask: '255.255.255.0', virtualbox__intnet: 'k8s-network'
      k8s_worker.vm.hostname = "k8s-worker#{i}"
    end
  end

  # Configuration du Client
  config.vm.define "client" do |client|
    client.vm.box = IMAGE_NAME
    client.vm.box_version = IMAGE_VERSION
    client.vm.synced_folder '.', '/vagrant', disabled: true

    client.vm.network "private_network", ip: "#{CLIENT_NETWORK_BASE}.10",
      netmask: '255.255.255.0', virtualbox__intnet: 'client-network'
    client.vm.hostname = "client"
  end

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "site.yml"
    ansible.groups = {
      "k8s-master" => ["k8s-master"],
      "k8s-worker" => ["k8s-worker[1:#{K8S_WORKER_NB}]"],
      "k8s-cluster:children" => ["k8s-master", "k8s-worker"]
    }
    ansible.extra_vars = {
      kubernetes_master_ip: "#{K8S_NODE_NETWORK_BASE}.10",
      kubernetes_worker_number: K8S_WORKER_NB,
      kubernetes_network_base: K8S_NODE_NETWORK_BASE,
      client_network_base: CLIENT_NETWORK_BASE,
      bgp_network_base: BGP_NETWORK_BASE
    }
  end
end