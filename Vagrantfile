IMAGE_NAME = "bento/ubuntu-18.04"
N = 2

Vagrant.configure("2") do |config|
    config.ssh.insert_key = false

    config.vm.provider "virtualbox" do |v|
        v.memory = 2048
        v.cpus = 2
    end
      
    config.vm.define "k8s-master" do |master|
        master.vm.box = IMAGE_NAME
        master.vm.network "private_network", ip: "192.168.52.10"
        master.vm.hostname = "k8s-master"
        master.vm.provision "ansible" do |ansible|
            ansible.playbook = "kubernetes-setup/master-playbook.yml"
        end
    end


# TODO: add second harddisk 
    (1..N).each do |i|

        config.vm.define "node-#{i}" do |node|
            node.vm.box = IMAGE_NAME
            node.vm.network "private_network", ip: "192.168.52.#{i + 10}"
            node.vm.hostname = "node-#{i}"

            node.vm.provider "virtualbox" do |v|
                v.memory = 2048
                v.cpus = 2
         
                file_to_disk="node-#{i}-sdb.vmdk"
                unless File.exist?(file_to_disk)
                    v.customize ['createhd', '--filename', file_to_disk, '--size', 50 * 1024]
                end
                v.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', file_to_disk]
            end

            node.vm.provision "ansible" do |ansible|
                ansible.playbook = "kubernetes-setup/node-playbook.yml"
            end
        end
    end
end
