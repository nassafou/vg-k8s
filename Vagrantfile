Vagrant.configure(2) do |config|
# Set some variables

        etcHosts = ""
	ingressNginx = ""
	wordpress = ""
	wordpressUrl = "wordpress.kub"

# Check ingress controller
	case ARGV[0]
		when "provision", "up"
  	print "Do you want nginx as ingress controller (y/n) ?\n"
  	ingressNginx = STDIN.gets.chomp
  	print "\n"

  if ingressNginx == "y"
	  print "Do you want a wordpress in your kubernetes cluster (y/n) ?\n"
  	wordpress = STDIN.gets.chomp
  	print "\n"

		# check if wordpress
  	if wordpress == "y"
 			print "Which url for your wordpress ?"
  		wordpressUrl = STDIN.gets.chomp
     	unless wordpressUrl.empty? then wordpressUrl else 'wordpress.url' end
		end
	end
	else
  	# do nothing
	end
# some settings for common server (not for haproxy)
  common = <<-SHELL
  sudo apt update -qq 2>&1 >/dev/null
  sudo apt install -y -qq git vim tree net-tools telnet git python3-pip sshpass nfs-common 2>&1 >/dev/null
  curl -fsSL https://get.docker.com -o get-docker.sh 2>&1
  sudo sh get-docker.sh 2>&1 >/dev/null
  sudo usermod -aG docker vagrant
  sudo service docker start
  sudo echo "autocmd filetype yaml setlocal ai ts=2 sw=2 et" > /home/vagrant/.vimrc
  sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/g' /etc/ssh/sshd_config
  sudo systemctl restart sshd
  SHELL



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
      #for haproxy
      if node[:type] == "haproxy"
	 cfg.vm.provision :shell, :path => "install_haproxy.sh"
      end
     
      # for all servers in cluster (need docker)
      if node[:type] == "kub"
	 cfg.vm.provision :shell, :inline => common
      end

      # for the deploy server
      if node[:type] == "deploy"
	 cfg.vm.provision :shell, :inline => common
	 cfg.vm.provision :shell, :path => "install_kubespray.sh", :args => ingressNginx
         if wordpress == "y"
            cfg.vm.provision :shell, :path => "install_nfs.sh"
            cfg.vm.provision :shell, :path => "install_wordpress.sh", :args => wordpressUrl
	end
      end
    end # end config
  end # end nodes
end 
