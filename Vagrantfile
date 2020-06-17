require 'dotenv/load'

vms = {
    "k8s-master"   => {
        :role => "master",
        :vm_box => "bento/centos-8.1", 
        :vm_box_version => "202005.21.0", 
        :vm_cpu => ENV['K8S_MASTER_CPU'], 
        :vm_ram => ENV['K8S_MASTER_RAM'], 
        :vm_ip => ENV['K8S_MASTER_IP']
    },
    "k8s-worker-1" => {
        :role => "worker",
        :vm_box => "bento/centos-8.1",
        :vm_box_version => "202005.21.0",
        :vm_cpu => ENV['K8S_WORKER_1_CPU'],
        :vm_ram => ENV['K8S_WORKER_1_RAM'],
        :vm_ip => ENV['K8S_WORKER_1_IP']
    }
}

Vagrant.configure("2") do |config|
    vms.each do | vm_name, vm_params |

        config.vm.define "#{vm_name}" do |vm_item|

            vm_item.vm.hostname    = "#{vm_name}"
            vm_item.vm.box         = "#{vm_params[:vm_box]}"
            vm_item.vm.box_version = "#{vm_params[:vm_box_version]}"

            vm_item.vm.network "private_network", ip: "#{vm_params[:vm_ip]}",
                virtualbox__intnet: true

            vm_item.vm.provider "virtualbox" do |vm_item_vb|

                vm_item_vb.cpus   = vm_params[:vm_cpu]
                vm_item_vb.memory = vm_params[:vm_ram]

            end
        end
    end

    config.vm.provision "ansible" do |ansible|
        ansible.compatibility_mode = "2.0"
        ansible.playbook = "playbooks/setup-tools.yml"
        ansible.become = true
        ansible.groups = {
            "setup_node" => ["k8s-master"]
        }
        ansible.extra_vars = {
            "kubectl_version" => "1.15.3"
        }
    end

    config.vm.provision "ansible" do |ansible|
        ansible.compatibility_mode = "2.0"
        ansible.playbook = "playbooks/setup-certificates.yml"
        ansible.become = true
        ansible.groups = {
            "setup_node" => ["k8s-master"]
        }
        ansible.extra_vars = {
            "kubernetes_master_nodes" => vms.select{ |k,v| v[:role] =~ /master/ }.map{ |k,v| {:host => k, :ip => v[:vm_ip]}},
            "kubernetes_worker_nodes" => vms.select{ |k,v| v[:role] =~ /worker/ }.map{ |k,v| {:host => k, :ip => v[:vm_ip]}},
            "certificates_c" => ENV['K8S_CA_C'],
            "certificates_l" => ENV['K8S_CA_L'],
            "certificates_o" => ENV['K8S_CA_O'],
            "certificates_ou" => ENV['K8S_CA_OU'],
            "certificates_st" => ENV['K8S_CA_ST'],
            "certificates_ca_expiry" => ENV['K8S_CA_EXPIRY']
        }
    end
end
