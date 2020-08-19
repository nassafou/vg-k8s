Vagrant.configure(2) do |config|

  etcHosts = ""

	config.vm.box = "ubuntu/bionic64"
	config.vm.box_url = "ubuntu/bionic64"

	# set servers list and their parameters
	NODES = [
  	{ :hostname => "autohaprox", :ip => "192.168.12.10", :cpus => 1, :mem => 512, :type => "haproxy" },
  	{ :hostname => "autokmaster", :ip => "192.168.12.11", :cpus => 4, :mem => 4096, :type => "kub" },
  	{ :hostname => "autoknode1", :ip => "192.168.12.12", :cpus => 2, :mem => 2048, :type => "kub" },
  	{ :hostname => "autodep", :ip => "192.168.12.20", :cpus => 1, :mem => 512, :type => "deploy" }
	]

	# define /etc/hosts for all servers
  NODES.each do |node|
    if node[:type] != "haproxy"
    	etcHosts += "echo '" + node[:ip] + "   " + node[:hostname] + "' >> /etc/hosts" + "\n"
		else
			etcHosts += "echo '" + node[:ip] + "   " + node[:hostname] + " autoelb.kub ' >> /etc/hosts" + "\n"
		end
  end #end NODES

	# run installation
  NODES.each do |node|
    config.vm.define node[:hostname] do |cfg|
			cfg.vm.hostname = node[:hostname]
      cfg.vm.network "private_network", ip: node[:ip]
      cfg.vm.provider "virtualbox" do |v|
				v.customize [ "modifyvm", :id, "--cpus", node[:cpus] ]
        v.customize [ "modifyvm", :id, "--memory", node[:mem] ]
        v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        v.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
        v.customize ["modifyvm", :id, "--name", node[:hostname] ]
      end #end provider
			
			#for all
      cfg.vm.provision :shell, :inline => etcHosts
    end # end config
  end # end nodes
end 
