Vagrant.configure("2") do |config|
	config.vm.box = "ubuntu/bionic64"
	#config.vm.network ="public_network", type: "dhcp" 
unless Vagrant.has_plugin?("vagrant-disksize")
	raise Vagrant::Errors::VagrantError.new, "vagrant-disksize plugin is missing. Please install it using 'vagrant plugin install vagrant-disksize' and rerun 'vagrant up'"
end
	config.disksize.size = '100GB'
	#config.ssh.forward_agent = true
	config.vm.provider "virtualbox" do |v|
		v.memory = 8192
		v.cpus	 = 4
		v.customize ["modifyvm", :id, "--nested-hw-virt", "on"]
	end
	
	$provisionscript = <<-SCRIPT
		apt-get update && apt-get upgrade -y
		apt-get install -y git build-essential curl pkg-config libssl-dev 
		apt-get install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils
		adduser vagrant libvirt
		adduser vagrant kvm
	SCRIPT
	$rustscript = <<-SCRIPT
		curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- --default-toolchain nightly -y
	SCRIPT
	$rustyhermit = <<-SCRIPT
		rustup default nightly
		cargo install cargo-download
		rustup component add rust-src
		rustup component add llvm-tools-preview
		cargo install uhyve
	SCRIPT
	$init_submodules = <<-SCRIPT
		cd /vagrant
		git submodule init
		git submodule update
	SCRIPT
	$check_nested_virtualization = <<-SCRIPT
		egrep -c '(vmx|svm)' /proc/cpuinfo
		ls -la /var/run/libvirt/libvirt-sock
		ls -l /dev/kvm
	SCRIPT
	config.vm.provision :shell, inline: $provisionscript
	config.vm.provision :shell, inline: $rustscript, privileged: false, reset: true
	config.vm.provision :shell, inline: $rustyhermit, privileged: false
	config.vm.provision :shell, inline: $init_submodules, privileged: false
	config.vm.provision :shell, inline: $check_nested_virtualization, privileged: true
	 
	
end