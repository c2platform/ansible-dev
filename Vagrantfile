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
    nd_name = "#{nodes_info['defaults']['prefix']}#{nd['name']}" # .e.g. c2dproxy
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
      end
      # cfg.vm.provision 'shell', inline: (nd['shell-provision'] || nodes_info['defaults']['shell-provision'])
      plays = ENV['PLAY'] ? [ENV['PLAY']] : nd['plays']
      plays.each do |p|
        cfg.vm.provision 'ansible' do |ansible|
          ansible.config_file = 'ansible-dev.cfg'
          ansible.playbook = ansible_play(p)
          ansible.compatibility_mode = '2.0'
          ansible.inventory_path = 'development.ini'
          # ansible.galaxy_role_file = '../roles/requirements.yml'
          # ansible.galaxy_roles_path = '~/.ansible/roles'
          # ansible.verbose = 'vvv'
        end
      end
    end
  end
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
  unless ENV['C2_REPO'].nil?
    url = File.join(ENV['C2_REPO'], "#{box_name}-#{box_version}.json")
    res = Net::HTTP.get_response(URI.parse(url.to_s))
    url unless res.code == '404'
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

# vpass file for Ansible vault secrets.yml
vpass_file = File.join(File.dirname(__FILE__), 'vpass')
File.open(vpass_file, 'w') { |f| f.write('secret') } unless File.exist? vpass_file
