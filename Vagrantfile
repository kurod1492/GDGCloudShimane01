# coding: utf-8
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  config.vm.network "private_network", ip: "192.168.33.10"
  # v1: Ruby をインストール
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get install -y build-essential libssl-dev
    apt-get install -y ruby-build
  SHELL

  config.vm.provision "shell", privileged: false, inline: <<-SHELL

  if ! [ -d .rbenv ]
  then
    git clone https://github.com/sstephenson/rbenv.git .rbenv
    git clone https://github.com/sstephenson/ruby-build.git .rbenv/plugins/ruby-build
    echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> .bashrc
    echo 'eval "$(rbenv init -)"' >> .bashrc
  fi

  SHELL

  # v2: install ruby by rbenv
  config.vm.provision "shell", privileged: false, inline: <<-SHELL
  
    if [ -e .rbenv/bin/rbenv ]
    then
      export PATH="$HOME/.rbenv/bin:$PATH"
      eval "$(rbenv init -)"

      rbenv install -s 2.6.4
      rbenv rehash
      rbenv global 2.6.4
    fi
  SHELL

  # v3: Docker をインストール
  config.vm.provision :shell, path: "dockerinstall.sh"
end
