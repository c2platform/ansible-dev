# frozen_string_literal: true

# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'openssl'
require 'yaml'
require 'net/http'

nodes_info = YAML.load_file('nodes.yml')

Vagrant.configure(2) do |config|
  config.vm.synced_folder '.', '/vagrant'
  nodes_info['nodes'].each do |nd|
    nd_name = "#{nodes_info['defaults']['prefix']}-#{nd['name']}"
    bx = nd['box'] || nodes_info['defaults']['box']
    bx_vsn = nd['version'] || nodes_info['defaults']['version']
    bx_url = box_url(bx, bx_vsn)
    config.vm.define nd_name do |cfg|
      cfg.vm.hostname = "#{nd_name}.bkd.local"
      cfg.vm.box = bx
      if bx_url
        cfg.vm.box_url = bx_url
        cfg.vm.box_version = bx_vsn
      end
      nd['ip-address'].split(',').each do |ip|
        cfg.vm.network 'private_network', ip: ip
      end
      cfg.vm.network 'public_network', bridge: nodes_info['defaults']['bridge'], mac: nd['mac'] if nd['mac']
      cfg.ssh.insert_key = false
      cfg.vm.provider 'virtualbox' do |vb|
        vb.gui = false
        vb.memory = nd['memory'] || 1024
        vb.cpus = nd['cpus'] || 2
        add_disk(vb, 1, 300 * 1024, "#{nd['name']}.vdi", 'IDE Controller') if nd['data-disk'] == true
        unless nd['trace'].nil?
          trace(vb,nodes_info['defaults']['trace-dir'],:id, nd_name, nd['trace'] )
        end
      end
      cfg.vm.provision 'shell', inline: (nd['shell-provision'] || nodes_info['defaults']['shell-provision'])
      plays = ENV['PLAY'] ? [ENV['PLAY']] : nd['plays']
      mnp = multinode_provision(plays)

      if !mnp || (nd_name == last_node)
        plays.each do |p|
          cfg.vm.provision 'ansible' do |ansible|
            ansible.config_file = 'ansible-dev.cfg'
            ansible.playbook = ansible_play(p)
            ansible.compatibility_mode = '2.0'
            ansible.inventory_path = 'development.ini'
            ansible.limit = limit if mnp
            # ansible.galaxy_role_file = '../roles/requirements.yml'
            # ansible.galaxy_roles_path = '~/.ansible/roles'
            # ansible.verbose = 'vvv'
          end
        end
      end
    end
  end
end

def trace(vb_provider, trace_dir, nd_id, nd_name, on_off = 'on', nic_no = 2)
  #puts "on_off: #{on_off.inspect}"
  vb_provider.customize ['modifyvm', nd_id, "--nictrace#{nic_no}", on_off, "--nictracefile#{nic_no}", "#{ trace_dir}/#{nd_name}.pcap"]
  puts "Trace #{nd_name} #{ trace_dir}/#{nd_name}.pcap" if on_off == "on"
end

def add_disk(vb_provider, port = 2, size = 120 * 1024, filename = nil, controller = 'IDE Controller')
  filename = "disk#{port}.vdi" if filename.nil?
  disk = File.expand_path(filename)
  vb_provider.customize ['createhd', '--filename', disk, '--format', 'VDI', '--size', size] unless File.exist?(disk)
  vb_provider.customize ['storageattach', :id, '--storagectl', controller, '--port', port, '--device', 0, '--type', 'hdd', '--medium', disk]
end

def vm_exists?(vm_name = 'default', provider = 'virtualbox')
  File.exist?(File.join(File.dirname(__FILE__), ".vagrant/machines/#{vm_name}/#{provider}/vagrant_cwd"))
end

def box_url(box_name, box_version)
  nodes_info = YAML.load_file('nodes.yml')
  if ENV['C2_VAGRANT_SERVER_URL']
    nodes_info['defaults']['repo'] = ENV['C2_VAGRANT_SERVER_URL']
  end
  url = File.join(nodes_info['defaults']['repo'], "#{box_name}-#{box_version}.json")
  unless url.nil?
    res = Net::HTTP.get_response(URI.parse(url.to_s))
    url unless res.code == '404'
  end
end

def limit
  if nodes_selected == []
    'all'
  else
    nodes_selected.join(':')
  end
end

def ansible_play(play)
  ps = ["./plays/#{play}.yml", "../ansible/plays/#{play}.yml"]
  ps2 = ps.collect { |pth| pth if File.exist?(pth) }.compact
  if ps2.empty?
    raise "Play #{play} not found! Paths #{ps.join(', ')} don't exists"
  else
    ps2.first
  end
end

# Return last node in command or the default multiprovison node
def last_node
  ns = nodes_selected
  if ns == []
    nodes_info = YAML.load_file('nodes.yml')
    nodes_info['defaults']['node_multinode_provision']
  else
    ns.last
  end
end

# Return nodes / boxes part of command
def nodes_selected
  nodes_info = YAML.load_file('nodes.yml')
  nodes_all = nodes_info['nodes'].collect { |n| "#{nodes_info['defaults']['prefix']}-#{n['name']}" }
  nodes_selected = []
  ARGV.each do |a|
    nodes_selected << a if nodes_all.include?(a)
  end
  nodes_selected
end

# Return true if play contains multiple hosts
def multinode_provision(plays)
  plays.each do |play|
    play_yml = File.read(ansible_play(play))
    return true if play_yml.scan(/import_playbook:\s?(\S+)/m).count > 1
    return true if play_yml.scan(/hosts:\s?(\S+)/m).count > 1
  end
  return false
end

# vpass file for Ansible vault secrets.yml
vpass_file = File.join(File.dirname(__FILE__), 'vpass')
File.open(vpass_file, 'w') { |f| f.write('secret') } unless File.exist? vpass_file
