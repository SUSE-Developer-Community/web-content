# Vagrant never gets old - 

TODO: fancier title

Vagrant is not new. In fact, it's so not new that articles providing a good entry level overview as well as the most important tips and tricks have become hard to come by. It's not one of those new shiny objects anymore, so nobody's going to read that article, right? 

Vagrant continues to be an extremely useful tool though. So let's get back to it for a short while and have a look what it has to offer. 

# What is Vagrant?

In a nutshell, Vagrant enables you to define virtual machines in a [declarative](https://en.wikipedia.org/wiki/Declarative_programming) way. This essentially means it gives you a way to automate the configuration steps you'd otheriwse need to perform manually after 

Tool to configure virtual machines through a declarative language. Think of it as a wrapper around your hypervisor. 

Draw a parallel to libvirt?

# The basics

## Installing Vagrant

TODO: Does it need to be set up for the hypervisor in use?

## Spinning up your first Vagrant box

vagrant init in an empty directory, edit Vagrant file. Start the box with Vagrant up. vagrant ssh into it.

Explain the different ways to turn Vagrant boxes on and off - suspend, resume, halt, etc.

# The next level

## Provisioning

How to install packages - shell, Ansible, Puppet, etc.

## Vagrant networking options

Explain port forwarding, private and public network, etc.

## Some more tips & tricks

Copy files in & out via /vagrant shared directory

How to get the config to use other ssh clients.  

## Choosing your Vagrant box image

What is there, how to find what you're looking for. Plug for openSUSE vagrant image. 

