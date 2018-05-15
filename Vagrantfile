Vagrant.configure(2) do |config|

    config.vm.box = "ubuntu/xenial64"
    config.vm.network "public_network"
    config.vm.provider "virtualbox" do |vb|
        ## Display the VirtualBox GUI when booting the machine
        #vb.gui = true

        vb.memory = "2048"
    end
    config.vm.define "default", primary: true do |web|
      # ...
    end
    (1..1).each do |i|
        config.vm.define "node-#{i}" do |node|
            # ...
        end
    end

end
