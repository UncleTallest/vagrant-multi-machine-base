# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"
Vagrant.configure("2") do |config|

  # MACHINE_TYPE = {
  #   :locker => 1,
  #   :mail =>1,
  #   :mon => 1,
  #   :services => 1,
  #   :solrcloud_master => 2,
  #   :solrcloud_replica => 1,
  #   :tracker => 1,
  #   :vpn => 1,
  #   :work => 1,
  #   :www => 3
  # }
  MACHINE_TYPE = {
    :vpn => 1
  }
  
  MACHINE_TYPE.each do |type,nodes|
    i = 1
    while i <= nodes
      node_index = "0#{i}"
      machine = "#{type}#{node_index}"
      machines = [machine]
      i+=1
      puts "The machines to build are: '#{machines}'"
      [ machines ].each do |system|
        config.vm.define machine do |sys|
          sys.vm.provider :aws do |aws, override|
            aws.box = "aws-basic"
            aws.access_key_id = ENV["AWS_ACCESS_KEY_ID"]
            aws.secret_access_key = ENV["AWS_SECRET_ACCESS_KEY"]
            aws.keypair_name ="INFRASTRUCTURE"
            aws.subnet_id ="subnet-47f74929"
            aws.availability_zone ="us-east-1d"
            aws.instance_type ="m1.small"
            # can be unique
            # TODO: how to get group id (required for VPC) from the group_name
            aws.security_groups = [ "sg-2d38ee42", "sg-e53fe18a", "sg-d9e23bb6" ]
            # can be unique
            aws.ami = "ami-4bb39522"
            # can be unique
            aws.elastic_ip = true 
            override.ssh.private_key_path = ENV["MY_PRIVATE_AWS_SSH_KEY_PATH"]
            # can be unique
            override.ssh.username = "ubuntu"
            override.ssh.timeout = 120
            override.ssh.max_tries = 10
            # override.ssh.host = :private_ip
            aws.tags = {
              "Name" => machine
            }
          end

          sys.omnibus.chef_version = :latest
          sys.berkshelf.enabled=true
          sys.vm.provision :chef_solo do |chef|
            chef.node_name = machine
            chef.cookbooks_path = "cookbooks"
            chef.add_recipe "apt"
            chef.add_recipe "locale-gen"
            chef.add_recipe "strongswan::ipsec"
            chef.add_recipe "strongswan::connections"
            chef.add_recipe "strongswan::routing"
            chef.add_recipe "vim"
            chef.roles_path = "roles"
            chef.add_role(sys == /vpn[0-9]/ ? 'vpn' : '')
            chef.json = {
              localegen:{
                lang:["en_US","en_US.utf8"]
              },
              strongswan: {
                ipsec: {
                  natt: "yes",
                  keyexchange: "ikev2",
                  scenarios: ["xauth-rsa-mode-config", "xauth-id-psk-mode-config"],
                  local: {
                    id: "server@strongswan.org",
                    subnet: "172.19.0.0/16"
                  },
                  remote: {
                    id: "client@strongswan.org",
                    sourceip: "172.19.101.0/24"
                  }
                }
              }
            }
          end
        end
      end
    end
  end
end
