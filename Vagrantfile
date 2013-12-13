# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  
  # Build a hash with all the machine types as keys and the desired number of each 
  # type of machine in an integer. 
  # MACHINE_TYPE = {
  #   :vpn => 1
  # }
  MACHINE_TYPE = { :hostname => 1 }

  # create an empty array to hold the hostnames we will generate next.
  machines = Array.new

  # Iterate through the hash and build hostnames from the keys and values; 
  # then push to the end of the empty array we just built.
  MACHINE_TYPE.each_pair do |key,value|
    i = 1
    until i > value.to_i
      node_index = "0#{i}"
      machine = "#{key}#{node_index}"
      machines.push(machine)
      i+=1
    end
  end
  
  # uncomment the puts statement below to see a list of buildable machines 
  # via the commandline:
  # [schade@metis ~/vagrant-multi-machine-base]$ vagrant status hostname01
  # The buildable machines are: '["hostname01"]'
  # Current machine states:

  # hostname01                not created (virtualbox)

  # The environment has not yet been created. Run `vagrant up` to
  # create the environment. If a machine is not created, only the
  # default provider will be shown. So if a provider is not listed,
  # then the machine is not created for that environment.
  # puts "The buildable machines are: '#{machines}'"

  # Now lets build the machines heldp within the array we just populated
  machines.each do |system|
    config.vm.define system do |sys|

      # We're using the vagant-aws plugin to build directly on EC2/VPC
      # vagrant plugin install vagrant-aws
      sys.vm.provider :aws do |aws, override|
        aws.box = "aws-basic"
        aws.access_key_id = ENV["AWS_ACCESS_KEY_ID"]
        aws.secret_access_key = ENV["AWS_SECRET_ACCESS_KEY"]
        aws.keypair_name ="<aws_keypair_name_goes_here>"
        aws.subnet_id ="<aws_subnet_id_goes_here>"
        aws.availability_zone ="<aws_availability_zone_goes_here>"
        aws.instance_type ="m1.small"

        # can be unique
        # TODO: how to get group id (required for VPC) from the group_name
        # For VPC you must use the Security Group ID number, for EC2 you 
        # may use the Security Group name. 
        aws.security_groups = [ 
          "<aws_group_id_goes_here>", 
          "<aws_group_id_goes_here>", 
          "<aws_group_id_goes_here>" 
        ]

        # can be unique; I tend to use ami-4bb39522 for my purposes.
        aws.ami = "<aws_ami_goes_here>"

        # can be unique
        aws.elastic_ip = true
        override.ssh.private_key_path = ENV["MY_PRIVATE_AWS_SSH_KEY_PATH"]
      
        # can be unique; ubuntu uses ubuntu redhat/centos use ec2-user
        override.ssh.username = "ubuntu"
        override.ssh.timeout = 120
        override.ssh.max_tries = 10

        # override.ssh.host = :private_ip
        aws.tags = {
          "Name" => system
        }
      end

      # Use latest version of whichever chef provisioner you chose to use
      # vagrant plugin install vagrant-omnibus
      sys.omnibus.chef_version = :latest

      # Please do yourself a favor and use berkshelf for cookbook resolution.
      # vagrant plugin install vagrant-berkshelf
      sys.berkshelf.enabled=true
      sys.vm.provision :chef_solo do |chef|
        chef.node_name = system
        chef.cookbooks_path = "cookbooks"
        chef.add_recipe "vim"
        # chef.add_role(sys == /hostname[0-9]/ ? 'hostname' : '')
        chef.json = {}
      end
    end
  end
end