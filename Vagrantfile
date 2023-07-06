# -*- mode: ruby -*-
# vim: set ft=ruby :
# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
:inetRouter => {
        :box_name => "centos/7",
        :box_version => "2004.01",
        #:public => {:ip => '10.10.10.1', :adapter => 1},
        :net => [
                   {ip: '192.168.255.1', adapter: 2, netmask: "255.255.255.248", virtualbox__intnet: "router-net"},
                ]
  },
  :inetRouter2 => {
    :box_name => "centos/7",
    :box_version => "2004.01",
    #:public => {:ip => '192.168.11.13', :adapter => 3},
    :net => [
               {ip: '192.168.255.3', adapter: 2, netmask: "255.255.255.248", virtualbox__intnet: "router-net"},
               {ip: '192.168.12.11', adapter: 3, netmask: "255.255.255.0"},
            ]
},
  :centralRouter => {
        :box_name => "centos/7",
        :box_version => "2004.01",
        :net => [
                   {ip: '192.168.255.2', adapter: 2, netmask: "255.255.255.248", virtualbox__intnet: "router-net"},
                   {ip: '192.168.0.1', adapter: 3, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},
                   {ip: '192.168.0.33', adapter: 4, netmask: "255.255.255.240", virtualbox__intnet: true},
                   {ip: '192.168.0.65', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: true},
                ]
  },
  
  :centralServer => {
        :box_name => "centos/7",
        :box_version => "2004.01",
        :net => [
                   {ip: '192.168.0.2', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},
                   {adapter: 3, auto_config: false, virtualbox__intnet: true},
                   {adapter: 4, auto_config: false, virtualbox__intnet: true},
                ]
  }
  
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

    config.vm.define boxname do |box|

        box.vm.box = boxconfig[:box_name]
        box.vm.host_name = boxname.to_s
        box.vm.provider :virtualbox do |vb|
          vb.customize ["modifyvm", :id, "--memory", "192"]
          vb.customize ["modifyvm", :id, "--cpus", "1"]
        end
        boxconfig[:net].each do |ipconf|
          box.vm.network "private_network", ipconf
        end
        
        if boxconfig.key?(:public)
          box.vm.network "public_network", boxconfig[:public]
        end

        box.vm.provision "shell", inline: <<-SHELL
          mkdir -p ~root/.ssh
                cp ~vagrant/.ssh/auth* ~root/.ssh
        SHELL
        
        case boxname.to_s
        when "inetRouter"
          box.vm.provision "shell", inline: <<-SHELL
          sysctl net.ipv4.conf.all.forwarding=1
          iptables -P INPUT ACCEPT
          iptables -P FORWARD ACCEPT
          iptables -P OUTPUT ACCEPT
          iptables -F
          iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
          # persistent settings
          echo "192.168.0.0/24 via 192.168.255.2 dev eth1" >> /etc/sysconfig/network-scripts/route-eth1
          echo "192.168.1.0/24 via 192.168.255.2 dev eth1" >> /etc/sysconfig/network-scripts/route-eth1
          echo "192.168.255.4/30 via 192.168.255.2 dev eth1" >> /etc/sysconfig/network-scripts/route-eth1
          echo "192.168.255.8/30 via 192.168.255.2 dev eth1" >> /etc/sysconfig/network-scripts/route-eth1
          # runtime settings
          ip route add 192.168.0.0/24 via 192.168.255.2 dev eth1
          ip route add 192.168.1.0/24 via 192.168.255.2 dev eth1
          ip route add 192.168.255.4/30 via 192.168.255.2 dev eth1
          ip route add 192.168.255.8/30 via 192.168.255.2 dev eth1

          iptables -P INPUT ACCEPT
          iptables -P FORWARD ACCEPT
          iptables -P OUTPUT ACCEPT
          iptables -F
          CHAINS="KNOCKING GATE1 GATE2 GATE3 PASSED"
          for chain in $CHAINS
            do
              iptables -N $chain
            done
            iptables -A INPUT -p tcp -i eth0 --dport 22 -j ACCEPT
            iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
            iptables -A INPUT -i lo -j ACCEPT
            iptables -A INPUT -j KNOCKING
            iptables -A GATE1 -p tcp --dport 1111 -m recent --name AUTH1 --set -j DROP
            iptables -A GATE1 -j DROP
            iptables -A GATE2 -m recent --name AUTH1 --remove
            iptables -A GATE2 -p tcp --dport 2222 -m recent --name AUTH2 --set -j DROP
            iptables -A GATE2 -j GATE1
            iptables -A GATE3 -m recent --name AUTH2 --remove
            iptables -A GATE3 -p tcp --dport 3333 -m recent --name AUTH3 --set -j DROP
            iptables -A GATE3 -j GATE1
            iptables -A PASSED -m recent --name AUTH3 --remove
            iptables -A PASSED -p tcp --dport 22 -j ACCEPT
            iptables -A PASSED -j GATE1
            iptables -A KNOCKING -m recent --rcheck --seconds 30 --name AUTH3 -j PASSED
            iptables -A KNOCKING -m recent --rcheck --seconds 10 --name AUTH2 -j GATE3
            iptables -A KNOCKING -m recent --rcheck --seconds 10 --name AUTH1 -j GATE2
            iptables -A KNOCKING -j GATE1
            
            
          SHELL
        when "inetRouter2"
          box.vm.provision "shell", inline: <<-SHELL
          # persistent settings
          sysctl net.ipv4.conf.all.forwarding=1
          echo "192.168.0.0/24 via 192.168.255.2 dev eth1" >> /etc/sysconfig/network-scripts/route-eth1
          echo "192.168.1.0/24 via 192.168.255.2 dev eth1" >> /etc/sysconfig/network-scripts/route-eth1
          echo "192.168.255.4/30 via 192.168.255.2 dev eth1" >> /etc/sysconfig/network-scripts/route-eth1
          echo "192.168.255.8/30 via 192.168.255.2 dev eth1" >> /etc/sysconfig/network-scripts/route-eth1
          # runtime settings
          ip route add 192.168.0.0/24 via 192.168.255.2 dev eth1
          ip route add 192.168.1.0/24 via 192.168.255.2 dev eth1
          ip route add 192.168.255.4/30 via 192.168.255.2 dev eth1
          ip route add 192.168.255.8/30 via 192.168.255.2 dev eth1
          iptables -A FORWARD -d 192.168.0.2/32 -p tcp -m tcp --dport 80 -m state --state NEW,RELATED,ESTABLISHED -j ACCEPT
          iptables -t nat -A PREROUTING -i eth2 -p tcp -m tcp --dport 8080 -j DNAT --to-destination 192.168.0.2:80
          iptables -t nat -A POSTROUTING -s 192.168.0.2/32 -o eth2 -p tcp -m tcp --sport 8080 -j SNAT --to-source 192.168.12.11
          SHELL
        when "centralRouter"
          box.vm.provision "shell", inline: <<-SHELL
            echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.d/netforward.conf
            sysctl --system > /dev/null
            # persistent settings
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.255.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            echo "DEVICE=eth3:1" >> /etc/sysconfig/network-scripts/ifcfg-eth3:1
            echo "IPADDR=192.168.255.6" >> /etc/sysconfig/network-scripts/ifcfg-eth3:1
            echo "NETMASK=255.255.255.252" >> /etc/sysconfig/network-scripts/ifcfg-eth3:1
            echo "192.168.2.0/24 via 192.168.255.5 dev eth3" > /etc/sysconfig/network-scripts/route-eth3
            echo "DEVICE=eth4:1" >> /etc/sysconfig/network-scripts/ifcfg-eth4:1
            echo "IPADDR=192.168.255.10" >> /etc/sysconfig/network-scripts/ifcfg-eth4:1
            echo "NETMASK=255.255.255.252" >> /etc/sysconfig/network-scripts/ifcfg-eth4:1
            echo "192.168.1.0/24 via 192.168.255.9 dev eth4" > /etc/sysconfig/network-scripts/route-eth4
            # runtime settings
            ip route delete default
            ip address add 192.168.255.6/30 dev eth3:1
            ip link set eth3:1 up
            ip address add 192.168.255.10/30 dev eth4:1
            ip link set eth4:1 up
            ip route add 192.168.1.0/24 via 192.168.255.9 dev eth4
            ip route add 192.168.2.0/24 via 192.168.255.5 dev eth3
            ip route add 0.0.0.0/0 via 192.168.255.1 dev eth1
            ip route add 192.168.12.0/24 via 192.168.255.3 dev eth1
            SHELL
        when "centralServer"
          box.vm.provision "shell", inline: <<-SHELL
          # persistent
          echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
          echo "GATEWAY=192.168.0.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
          # runtime
          ip route delete default
          ip route add default via 192.168.0.1 dev eth1
          yum -y install epel-release
          yum -y install nginx
          systemctl enable nginx
          systemctl start nginx
          SHELL
        end

      end

  end
  
  
end
