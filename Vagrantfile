# -*- mode: ruby -*-
# vi: set ft=ruby :


$common=<<-SCRIPT
	sudo apt update
	sudo apt upgrade -y
	sudo sed -i /etc/ssh/sshd_config -e 's/^.*LoginGraceTime.*$/LoginGraceTime 10s/g'
	sudo sed -i /etc/ssh/sshd_config -e 's/^.*MaxAuthTries.*$/MaxAuthTries 2/g'
	sudo sed -i /etc/ssh/sshd_config -e 's/^.*MaxSessions.*$/MaxSessions 2/g'
	sudo sed -i /etc/ssh/sshd_config -e 's/^.*PermitEmptyPasswords.*$/PermitEmptyPasswords no/g'
	sudo sed -i /etc/ssh/sshd_config -e 's/^.*HostbasedAuthentication.*$/HostbasedAuthentication no/g'
	sudo sed -i /etc/ssh/sshd_config -e 's/^.*PasswordAuthentication.*$/PasswordAuthentication no/g'
	sudo sed -i /etc/ssh/sshd_config -e 's/^.*PubkeyAuthentication.*$/PubkeyAuthentication yes/g'
	sudo sed -i /etc/ssh/sshd_config -e 's/^.*PermitRootLogin.*$/PermitRootLogin no/g'
	sudo echo "Protocol 2" >> /etc/ssh/sshd_config
	sudo systemctl restart sshd
	sudo usermod -p `openssl passwd 123456ubuntu` ubuntu
	#for ((i=1; i<4;i++)); do sudo bash -c "echo 192.168.25.$i vm$i >> /etc/hosts" ; done
SCRIPT

# 1st VM
$vm1=<<-SCRIPT1
	sudo apt install ubuntu-desktop-minimal chromium-browser xrdp -y
	sudo apt install gnome -y
	sudo snap install snap-store
	sudo adduser xrdp ssl-cert
	sudo service xrdp start
	#sudo su - ubuntu bash -c "ssh-keygen -q -t rsa -N '' -f ~/.ssh/id_rsa <<<y >/dev/null 2>&1"
SCRIPT1

# 2nd VM
$vm2=<<-SCRIPT2
	sudo mkdir -m 0755 -p /etc/apt/keyrings
	curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
	echo \
		"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
		$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
	sudo apt update
	sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
	sudo apt install wireguard-tools -y
	git clone https://github.com/ngoduykhanh/wireguard-ui
	cd wireguard-ui
	sudo docker compose up -d
SCRIPT2

# 3rd VM
$vm3=<<-SCRIPT3
	sudo useradd -d /home/adam -s /bin/bash -U -m adam
	sudo usermod -a -G sudo adam
	sudo usermod -p `openssl passwd adam` adam
SCRIPT3

	#id_rsa_ssh_key_pub = File.read(File.join(Dir.home, ".ssh", "id_rsa.pub"))
	system( "ssh-keygen -f id_rsa -t rsa -N ''", [ :out, :err ] => File::NULL) unless File.exists?("id_rsa") #win10
	id_rsa_ssh_key = File.read("id_rsa")
	id_rsa_ssh_key_pub = File.read("id_rsa.pub")
	File.delete("id_rsa")
	File.delete("id_rsa.pub")
 
$keys=<<-KEYS
	sudo bash -c \"echo '#{id_rsa_ssh_key_pub }' > /home/ubuntu/.ssh/authorized_keys\" && sudo chmod 600 /home/ubuntu/.ssh/authorized_keys && sudo chown ubuntu: /home/ubuntu/.ssh/authorized_keys
	sudo bash -c \"echo '#{id_rsa_ssh_key }' > /home/ubuntu/.ssh/id_rsa\" && sudo chmod 600 /home/ubuntu/.ssh/id_rsa && sudo chown ubuntu: /home/ubuntu/.ssh/id_rsa
	sudo bash -c \"echo '#{id_rsa_ssh_key }' > /home/vagrant/.ssh/id_rsa\" && sudo chmod 600 /home/vagrant/.ssh/id_rsa && sudo chown vagrant: /home/vagrant/.ssh/id_rsa
KEYS

Vagrant.configure("2") do |config|
	# documentation at https://docs.vagrantup.com.
	# boxes at https://vagrantcloud.com/search.
  
	config.vm.box = "jammy"
	config.vm.box_check_update = false
	config.vbguest.auto_update = false
	config.vm.synced_folder ".", "/vagrant", disabled: true
	config.vm.provision "shell", inline: $common
	#config.vm.network "forwarded_port", guest: 80, host: 8080, auto_correct: true

	(1..3).each do |i|
		config.vm.define "vm0#{i}" do |node|
			node.vm.hostname = "vm#{i}"
			node.vm.network "private_network", ip: "192.168.25.#{i}", netmask: "16", :name => 'VirtualBox Host-Only Ethernet Adapter', :adapter => 2
			
			if "#{i}" == "1"
				node.vm.provision "shell", inline: $vm1
				# for xrdp
				node.vm.network "forwarded_port", guest: 3389, host: 3390, auto_correct: true, host_ip: "127.0.0.1"
			end
			if "#{i}" == "2"
				node.vm.provision "shell", inline: $vm2
				# for wireguard-ui
				node.vm.network "forwarded_port", guest: 5000, host: 5001, auto_correct: true, host_ip: "127.0.0.1"
			end
			if "#{i}" == "3"
				node.vm.provision "shell", inline: $vm3
			end
			
			node.vm.provision :shell, :inline => $keys
			node.vm.provider "virtualbox" do |vb|
				#vb.gui = true
				vb.linked_clone = true
				vb.memory = 2048
				vb.cpus = 2
				vb.name = "vm#{i}"
			end
		end
	end
end
