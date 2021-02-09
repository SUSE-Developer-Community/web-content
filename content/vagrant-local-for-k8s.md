# Vagrant never gets old 

Let's face it -- Vagrant is not exactly a shiny new object. In fact, it's so not new that there are few articles providing an entry-level overview as well as important tips and tricks. So what's the point in writing about Vagrant in 2021? 

It turns out that Vagrant is still an extremely useful tool. In this article, I'll give a brief overview of what it has to offer. Sure, you could figure all this out by reading the documentation and playing around a bit. But if you're trying to get a job done (that does not happen to be learning Vagrant in the first place), this article might get you there faster. 

## What is Vagrant?

In a nutshell, Vagrant lets you define virtual machines in a [declarative](https://en.wikipedia.org/wiki/Declarative_programming) way. It gives you a way to keep the configuration of your VM, including memory, disk and CPU needs, as well as any possible post install provisioning and configuration steps in a file so that it is easily reproducible. Otherwise, you'd need to perform this configuration manually each time you spin up a new VM from an installation image. Over time, getting the exact same result would be tricky.

In addition, Vagrant gives you a convenient way to start, suspend, resume and update your VMs without having to interact with your hypervisor directly. 
TODO: walk through the steps top spin up a VM without Vagrant to illustrate the difference? 

In a nutshell, think of Vagrant as a wrapper around your hypervisor. 

## The Basics of Vagrant

### Installing Vagrant

Hashicorp provides Vagrant installers for all major platforms on their [download page](https://www.vagrantup.com/downloads). As per their [installation instructions](https://learn.hashicorp.com/tutorials/vagrant/getting-started-install) these installers are preferred over packaged versions available in upstream package repositories. Note that I installed it via Homebrew and haven't hit any problems yet. I'm also confident that the Vagrant package on openSUSE Leap works just fine - you may not have the very latest version but that's probably fine for most people. 

If you want to use Vagrant with a Hypervisor that is not installed by default, you should install that one first. In my case, I'm using Vagrant on top of Oracle [Virtualbox](https://www.virtualbox.org) on a Mac. 

### Spinning up your first Vagrant box

To get going with a fresh new Vagrant box, you need to run vagrant init in an empty directory. There's always one Vagrant box per directory. After vagrant init, you'll have a default Vagrantfile in that directory, which you can take as the starting point. You can of course simply overwrite this if you already have a working configuration somewhere else. 

Let's take a look at this file to see what's there. Note that I've trimmed away some of the comments since Vagrant tries to be _very_ self-explanatory. And I'm the one doing the explaining here. A side note: the default Vagrant file does of course only contain the most common parameters. A complete reference can be found at [https://docs.vagrantup.com](https://docs.vagrantup.com).

    # All Vagrant configuration is done below. The "2" in Vagrant.configure
    # configures the configuration version (we support older styles for
    # backwards compatibility). Please don't change it unless you know what
    # you're doing.
    Vagrant.configure("2") do |config|
    # Every Vagrant development environment requires a box. You can search for
    # boxes at https://vagrantcloud.com/search.
    config.vm.box = "opensuse/Leap-15.2.x86_64"

    # Disable automatic box update checking.
    # config.vm.box_check_update = false

    # Create a forwarded port mapping
    # NOTE: This will enable public access to the opened port
    # config.vm.network "forwarded_port", guest: 80, host: 8080

    # Create a forwarded port mapping which allows access to a specific port
    # within the machine from a port on the host machine and only allow access
    # via 127.0.0.1 to disable public access
    # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

    # Create a private network, which allows host-only access to the machine
    # using a specific IP.
    config.vm.network "private_network", ip: "192.168.33.10"

    # Create a public network, which generally matched to bridged network.
    # Bridged networks make the machine appear as another physical device on
    # your network.
    # config.vm.network "public_network"

    # Share an additional folder to the guest VM.
    # config.vm.synced_folder "../data", "/vagrant_data"

    # Provider-specific configuration so you can fine-tune various
    # backing providers for Vagrant. These expose provider-specific options.
    # Example for VirtualBox:
    #
    # config.vm.provider "virtualbox" do |vb|
    #   # Display the VirtualBox GUI when booting the machine
    #   vb.gui = true
    #
    #   # Customize the amount of memory on the VM:
    #   vb.memory = "1024"
    # end

    # Enable provisioning with a shell script. Additional provisioners such as
    # Ansible, Chef, Docker, Puppet and Salt are also available.
    # config.vm.provision "shell", inline: <<-SHELL
    #   apt-get update
    #   apt-get install -y apache2
    # SHELL
    end

Like with any VM, there are five important parts here: 
* Base image declaration: this defines the starting base VM image. I'm choosing my favorite Linux distribution, openSUSE Leap, in its current version 15.2. More on Vagrant base boxes later. 
* Network config: how to reach your Vagrant box. As you can see, you can choose between exposing specific ports (which can be limited to being accessible only from certain IP addresses), exposing a host-only IP or bridging the guest's virtual network interface to the host's external network so that it becomes visible to other machines in the same network. Obviously, the latter needs to be handled with care. Don't forget to turn off your VMs before heading off to your next conference (whenever that is). I'm going with a private IP for now. 
* Shared folders for data exchange between guest and host: by default, Vagrant mounts the directory where your vagrantfile is to /vagrant in the guest, which is perfectly fine for copying the occasional config file around and similar things. 
* Provider config: where you define your hypervisor settings, such as how much memory your VM should have
* Provisioning: [TO DO include definition] (e.g. install security updates or extra packages post-install). The example shown here uses the so-called shell provisioner, but you can also use proper provisioning languages like Ansible, Salt, etc.

Now that we have put together a basic configuration, let's take our new box out for a spin. All we need to do is run a simple `vagrant up` in the vagrant file directory. 

    > vagrant up
    Bringing machine 'default' up with 'virtualbox' provider...
    ==> default: Checking if box 'opensuse/Leap-15.2.x86_64' version '15.2.31.309' is up to date...
    ==> default: A newer version of the box 'opensuse/Leap-15.2.x86_64' for provider 'virtualbox' is
    ==> default: available! You currently have version '15.2.31.309'. The latest is version
    ==> default: '15.2.31.338'. Run `vagrant box update` to update.
    ==> default: Clearing any previously set forwarded ports...
    ==> default: Clearing any previously set network interfaces...
    ==> default: Preparing network interfaces based on configuration...
        default: Adapter 1: nat
    ==> default: Forwarding ports...
        default: 22 (guest) => 2222 (host) (adapter 1)
    ==> default: Booting VM...
    ==> default: Waiting for machine to boot. This may take a few minutes...
        default: SSH address: 127.0.0.1:2222
        default: SSH username: vagrant
        default: SSH auth method: private key
        default: Warning: Connection reset. Retrying...
        default: 
        default: Vagrant insecure key detected. Vagrant will automatically replace
        default: this with a newly generated keypair for better security.
        default: 
        default: Inserting generated public key within guest...
        default: Removing insecure key from the guest if it's present...
        default: Key inserted! Disconnecting and reconnecting using new SSH key...
    ==> default: Machine booted and ready!
    ==> default: Checking for guest additions in VM...
    ==> default: Mounting shared folders...
        default: /vagrant => /Users/tim/Documents/src/vagrant/test

Vagrant checks to see if it has the specified image available locally (Vagrant caches VM images to reduce the amount of data it needs to download), downloads the image if it isn't available in the local cache, brings up the VM, injects an ssh key and mounts the specified shared folders for easy data exchange between guest and host. 

Now we can use `vagrant ssh` to connect to the console. The Vagrant box can also be reached from the host via the private network IP address set in the vagrantfile. If you prefer to use your own ssh client, you can get all necessary information with the command `vagrant ssh-config`. 

Turning a Vagrant box off again can be done with `vagrant suspend` or `vagrant halt` if you want to basically power it off. If you use `vagrant suspend` you can resume the box with `vagrant resume` and the machine will have preserved its state. If you halted the machine, another `vagrant up` will bring it back. 

If you don't need your box anymore, you can run `vagrant destroy` to get rid of it entirely, including the VM disk volumes, which can occupy a lot of storage space. Be aware that this works on running boxes, too, which could mean an unexpected end to your day. 

If you ever need to change your vagrantfile, you can update a box configuration with `vagrant reload`. Vagrant can check your vagrantfile for syntax error with `vagrant validate`. If you want to run the provisioning part of your vagrantfile only, you can do this with `vagrant provision`. 

## (Auto)provisioning

Having a declarative way of defining the VM configuration (instead of having to sift through Virtualbox's menus) is great. But Vagrant's real power is its ability to do almost any kind of post-install provisioning and configuration. No matter how comprehensive your list of manual setup tasks, with Vagrant you only have to write it down once. Then you'll get 100 percent identical VMs every time you spin up a box. If something goes wrong, just throw it away and start over in a few minutes. Cattle vs. pets at its best, right? 

Let's look at an example. One thing you'll want to do with any new VM you're spinning up is to make sure you have all updates and security fixes installed that popped up after you built your image. Maybe there are also a couple of packages you want to install to tailor the VM according to your use case and taste. Let's further imagine I want to spin up a small experimental Kubernetes cluster in my VM. My go-to solution is [K3s] (https://k3s.io). One of the many great things about K3s is that it installs with a simple command. To access the K3s cluster within the VM from the host, I'll have to copy out the cluster configuration file to the host and point kubectl to this config. To do all this I simply need to add the following lines to the vagrantfile: 

  config.vm.provision "shell", inline: <<-SHELL
    zypper update
    zypper --non-interactive in command-not-found nano tmux 
    zypper --non-interactive in gnu-netcat curl
    curl -sfL https://get.k3s.io | sh -
    export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
    cp /etc/rancher/k3s/k3s.yaml /vagrant/
  SHELL

### Vagrant Cloud and a Few Tips for Choosing your Base Image

The base image you declare via `config.vm.box` in the vagrant file comes from a cloud service called [Vagrant Cloud](https://app.vagrantup.com/boxes/search). Essentially, Vagrant Cloud is to VMs what Docker hub is to containers - a salad bowl of all sorts of VM images. Some of them are generic base boxes, while some are tailored toward specific use cases by means of specific configuration and selection (addition and removal) of software contained in the image. You'll find boxes containing the base operating systems of all major Linux distributions such as Ubuntu, CentOS and, of course, openSUSE. 

Note that the same distro and version are sometimes provided by more than one account. Everyone can create an account on Vagrant Cloud and name it whatever they want, if the name is available. For example, the official Vagrant images maintained by the openSUSE community can be found under [https://app.vagrantup.com/opensuse](https://app.vagrantup.com/opensuse). 

However, if you just search for "opensuse", your first hit will likely be an image called `generic/opensuse15`, which happens to show up first in the list because it has more downloads than the official image. This image is done by the [Roboxes project](https://roboxes.org), which provides a variety of virtual machine images built with Hashicorp's [Packer](https://www.packer.io) tool. While this is certainly a credible source, but, I'd rather go for the original, particularly since it's not entirely clear which openSUSE [flavor](https://software.opensuse.org) this image actually contains. There's openSUSE Leap, Tumbleweed, Kubic, etc. By inference I would guess the `generic/opensuse15` is based on openSUSE Leap, since Leap's current major version is indeed 15. But is it 15.0, 15.1 or 15.2? Too much headache for my taste...

Each image is specific to a certain so-called provider, which essentially means hypervisor. So if you're using Virtualbox as the underlying virtualization provider, you need to make sure your chosen image supports that provider. One neat feature of Vagrant Cloud is that there are two little tabs on each image's page titles: "vagrantfile" and "new". Vagrantfile gives you the lines you need to copy paste into your - you guessed it - vagrantfile to use this image, and the new tab gives you the commands to init and spin a box up with this image in one go.

## Managing the Local Image Cache

As we did in our example above, when you specify the Vagrant image without declaring a specific version like

   config.vm.box = "opensuse/Leap-15.2.x86_64"

you will start seeing a message like this a few days after you've downloaded the image for the first time:

    > vagrant up
    Bringing machine 'default' up with 'virtualbox' provider...
    ==> default: Checking if box 'opensuse/Leap-15.2.x86_64' version '15.2.31.309' is up to date...
    ==> default: A newer version of the box 'opensuse/Leap-15.2.x86_64' for provider 'virtualbox' is
    ==> default: available! You currently have version '15.2.31.309'. The latest is version
    ==> default: '15.2.31.338'. Run `vagrant box update` to update.

This is because the images in Vagrant Cloud are being kept up to date by the openSUSE project and a new version comes out every couple of days. 

Let's take a look at the locally cached images on my machine at the time of this writing:

    > vagrant box list
    opensuse/Leap-15.2.x86_64  (virtualbox, 15.2.31.155)
    opensuse/Leap-15.2.x86_64  (virtualbox, 15.2.31.163)
    opensuse/Leap-15.2.x86_64  (virtualbox, 15.2.31.309)
    opensuse/Tumbleweed.x86_64 (virtualbox, 1.0.20200810)
    opensuse/Tumbleweed.x86_64 (virtualbox, 1.0.20200824)

As you can see, I have a couple of different versions of openSUSE Leap and Tumbleweed. The `vagrant box` command and its subcommands give you a couple of ways to manage the local image cache. As suggested by the output above, you can get the latest version of an image with `vagrant box update`. You can remove an image with `vagrant box remove` and you can add an image you downloaded manually or created yourself with `vagrant box add`. The latter is also the key to being able to verify the integrity of the images in your image cache with e.g. a sha256 checksum, since you need to wrap the checksum in a little bit of JSON before you can do that. I'll get more into that in a later post, but you can find some info in this [a bug report](https://github.com/CentOS/sig-cloud-instance-build/issues/118) on the CentOS GitHub page. 

## Famous Last Words

This is all for today - hope this was useful. If you liked the article, please cheer it so that it becomes easier for others to find. You're also highly welcome to leave a comment, ask a follow-up question or tell me what I got wrong in the comments. What is your favorite Vagrant command I didn't cover? 
