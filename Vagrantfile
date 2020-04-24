Vagrant.configure("2") do |config|
	config.vm.box = "ubuntu/bionic64"
	#config.vm.network ="public_network", type: "dhcp" 
unless Vagrant.has_plugin?("vagrant-disksize")
	raise Vagrant::Errors::VagrantError.new, "vagrant-disksize plugin is missing. Please install it using 'vagrant plugin install vagrant-disksize' and rerun 'vagrant up'"
end
	config.disksize.size = '100GB'
	config.ssh.forward_agent = true
	config.vm.provider "virtualbox" do |v|
		v.memory = 8192
		v.cpus	 = 4
		v.customize ["modifyvm", :id, "--nested-hw-virt", "on"]
	end
	
	$provisionscript = <<-SCRIPT
		apt-get update && apt-get upgrade -y
		apt-get install -y git build-essential curl pkg-config libssl-dev 
		apt-get install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils
		apt-get install -y python
		adduser vagrant libvirt
		adduser vagrant kvm
		bash -c "$(wget -O - https://apt.llvm.org/llvm.sh)"
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
		echo "github.com ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAq2A7hRGmdnm9tUDbO9IDSwBK6TbQa+PXYPCPy6rbTrTtw7PHkccKrpp0yVhp5HdEIcKr6pLlVDBfOLX9QUsyCOV0wzfjIJNlGEYsdlLJizHhbn2mUjvSAHQqZETYP81eFzLQNnPHt4EVVUh7VfDESU84KezmD5QlWpXLmvU31/yMf+Se8xhHTvKSCZIFImWwoG6mbUoWf9nzpIoaSjB+weqqUUmpaaasXVal72J+UX2B+2RPW3RcT0eOzQgqlJL3RKrTJvdsjE3JEAvGq3lGHSZXy28G3skua2SmVi/w4yCE6gbODqnTWlg7+wC604ydGXA8VJiS5ap43JXiUFFAaQ==" >> ~/.ssh/known_hosts
		git clone git@github.com:jschwe/hermit.git
		cd hermit
		git submodule init
		git submodule update
	SCRIPT
	$check_nested_virtualization = <<-SCRIPT
		egrep -c '(vmx|svm)' /proc/cpuinfo
		ls -la /var/run/libvirt/libvirt-sock
		ls -l /dev/kvm
	SCRIPT
	$build_rustc = <<-SCRIPT
        cd hermit/rust
        cp /vagrant/config.toml >> ./config.toml
        ./x.py build
        rustup toolchain link stage2 build/x86_64-unknown-linux-gnu/stage2
	SCRIPT
	config.vm.provision :shell, inline: $provisionscript
#	config.vm.provision :shell, inline: $rustscript, privileged: false, reset: true
#	config.vm.provision :shell, inline: $rustyhermit, privileged: false
	config.vm.provision :shell, inline: $init_submodules, privileged: false
	config.vm.provision :shell, inline: $build_rustc, privileged: false
	config.vm.provision :shell, inline: $check_nested_virtualization, privileged: true

	 
	
end
