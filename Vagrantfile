# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

#
# Vagrant confiugre
#
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

%w{
    centos-6.5
    centos-7.0
    centos-5.10
  }.each_with_index do |platform, index|

    config.vbguest.auto_update = false
    config.vm.define platform do |c|

      c.vm.box = "opscode-#{platform}"
      c.vm.box_url = "http://opscode-vm-bento.s3.amazonaws.com/vagrant/virtualbox/opscode_#{platform}_chef-provisionerless.box"
      c.vm.hostname = "vagrant4omnibus-itamae-#{platform}"
      c.vm.network :private_network, ip:"192.168.56.18#{index}"

      case platform
      when 'centos-5.10'
        epel = "rpm -Uvh http://ftp-srv2.kddilabs.jp/Linux/distributions/fedora/epel/5/x86_64/epel-release-5-4.noarch.rpm"
      when 'centos-6.5'
        epel = "yum -y install http://ftp.riken.jp/Linux/fedora/epel/6/x86_64/epel-release-6-8.noarch.rpm"
      when 'centos-7.0'
        epel = "yum -y install epel-release"
      else
        raise "Unknown platform: #{platform}"
      end

      c.vm.provision :shell, :inline => <<-EOT
        # epel repo
        #{epel}

        # packages
        yum -y install \
          git \
          openssl-devel \
          make \
          bzip2 \
          autoconf \
          automake \
          libtool \
          bison \
          gcc-c++ \
          patch \
          readline \
          readline-devel \
          zlib \
          zlib-devel \
          libffi-devel \
          gdbm-devel \
          libxslt-devel \
          libxml2-devel \
          fakeroot \
          rpm-build

        # rbenv and ruby
        yum -y remove ruby
        git clone https://github.com/sstephenson/rbenv.git ~/.rbenv
        echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
        echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
        cd
        source .bash_profile
        git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
        rbenv install 2.2.1
        rbenv global 2.2.1

        # gem
        gem install bundler

        # omnibus itamae
        git clone https://github.com/itamae-kitchen/omnibus-itamae.git
        cd omnibus-itamae/
        bundle install --binstubs

        # run omnibus build
        bin/omnibus build itamae

        # copy created package to shared directroy
        mkdir -p /vagrant/#{platform}
        cp -pr pkg/* /vagrant/#{platform}/

      EOT

      c.vm.provider :virtualbox do |vb|
        vb.gui = false
        vb.customize ["modifyvm", :id, "--memory", "2048"]
      end

    end

  end

end
