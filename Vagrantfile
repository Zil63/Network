# -*- mode: ruby -*-
# vim: set ft=ruby :
MACHINES = {
    :inetRouter => {
        :box_name => "centos/7",
        :vm_name => "inetRouter",
        #:public => {:ip => '10.10.10.1', :adapter => 1},
        :net => [
            {ip: '192.168.255.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
            {ip: '192.168.56.10', adapter: 8},
        ]
    },
    :centralRouter => {
        :box_name => "centos/7",
        :vm_name => "centralRouter",
        :net => [
            {ip: '192.168.255.2', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
            {ip: '192.168.0.1', adapter: 3, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},
            {ip: '192.168.0.33', adapter: 4, netmask: "255.255.255.240", virtualbox__intnet: "hw-net"},
            {ip: '192.168.0.65', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "mgt-net"},
            {ip: '192.168.255.9', adapter: 6, netmask: "255.255.255.252", virtualbox__intnet: "office1-central"},
            {ip: '192.168.255.5', adapter: 7, netmask: "255.255.255.252", virtualbox__intnet: "office2-central"},
            {ip: '192.168.56.11', adapter: 8},            
        ]
    },
    :centralServer => {
        :box_name => "centos/7",
        :vm_name => "centralServer",
        :net => [
            {ip: '192.168.0.2', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},
            {adapter: 3, auto_config: false, virtualbox__intnet: true},
            {adapter: 4, auto_config: false, virtualbox__intnet: true},
            {ip: '192.168.56.12', adapter: 8},
        ]
    },
    :office1Router => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "office1Router",
        :net => [
            {ip: '192.168.255.10', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "office1-central"},
            {ip: '192.168.2.1', adapter: 3, netmask: "255.255.255.192", virtualbox__intnet: "dev1-net"},
            {ip: '192.168.2.65', adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "test1-net"},
            {ip: '192.168.2.129', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "managers-net"},
            {ip: '192.168.2.193', adapter: 6, netmask: "255.255.255.192", virtualbox__intnet: "office1-net"},
            {ip: '192.168.56.20', adapter: 8},
        ]
    },
    :office1Server => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "office1Server",
        :net => [
            {ip: '192.168.2.130', adapter: 2, netmask: "255.255.255.192", virtualbox__intnet: "managers-net"},
            {ip: '192.168.56.21', adapter: 8},
        ]
    },
    :office2Router => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "office2Router",
        :net => [
            {ip: '192.168.255.6', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "office2-central"},
            {ip: '192.168.1.1', adapter: 3, netmask: "255.255.255.128", virtualbox__intnet: "dev2-net"},
            {ip: '192.168.1.129', adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "test2-net"},
            {ip: '192.168.1.193', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "office2-net"},
            {ip: '192.168.56.30', adapter: 8},
        ]
    },
    :office2Server => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "office2Server",
        :net => [
            {ip: '192.168.1.2', adapter: 2, netmask: "255.255.255.128", virtualbox__intnet: "dev2-net"},
            {ip: '192.168.56.31', adapter: 8},
        ]
    }    
}

Vagrant.configure("2") do |config|
    MACHINES.each do |boxname, boxconfig|
        config.vm.define boxname do |box|
            
            box.vm.box = boxconfig[:box_name]
            box.vm.host_name = boxname.to_s

            if boxconfig[:vm_name] == "office2Server"
                box.vm.provision "ansible" do |ansible|
                    ansible.playbook = "ansible/provision.yml"
                    ansible.inventory_path = "ansible/hosts"
                    ansible.host_key_checking = "false"
                    ansible.limit = "all"
                end
            end

            boxconfig[:net].each do |ipconf|
                box.vm.network "private_network", **ipconf
              end
              
              if boxconfig.key?(:public)
                box.vm.network "public_network", boxconfig[:public]
              end

            box.vm.provision "shell", inline: <<-SHELL
                mkdir -p ~root/.ssh
                cp ~vagrant/.ssh/auth* ~root/.ssh
            SHELL

            # case boxname.to_s
            # when "inetRouter"
            #     box.vm.provision "shell", run: "always", inline: <<-SHELL
            #         sysctl net.ipv4.conf.all.forwarding=1
            #         iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
            #     SHELL
            # when "centralRouter"
            #     box.vm.provision "shell", run: "always", inline: <<-SHELL
            #         sysctl net.ipv4.conf.all.forwarding=1
            #         echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
            #         echo "GATEWAY=192.168.255.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            #         systemctl restart network
            #     SHELL
            # when "centralServer"
            #     box.vm.provision "shell", run: "always", inline: <<-SHELL
            #         echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
            #         echo "GATEWAY=192.168.0.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            #         systemctl restart network
            #     SHELL
            # end
        end
    end
end
