$logger = Log4r::Logger.new('vagrantfile')
def read_ip_address(machine)
  command =  "ip a | grep 'inet' | grep -v '127.0.0.1' | cut -d: -f2 | awk '{ print $2 }' | cut -f1 -d\"/\""
  result  = ""

  $logger.info "Processing #{ machine.name } ... "

  begin
    # sudo is needed for ifconfig
    machine.communicate.sudo(command) do |type, data|
      result << data if type == :stdout
    end
    $logger.info "Processing #{ machine.name } ... success"
  rescue
    result = "# NOT-UP"
    $logger.info "Processing #{ machine.name } ... not running"
  end

  # the second inet is more accurate
  result.chomp.split("\n").select { |hash| hash != "" }[1]
end


Vagrant.configure("2") do |config|
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.manage_guest = true
  config.hostmanager.ignore_private_ip = false

  if Vagrant.has_plugin?("HostManager")
    config.hostmanager.ip_resolver = proc do |vm, resolving_vm|
      read_ip_address(vm)
    end
  end

  config.vm.define "server-log" do |server_log|
      server_log.vm.box = 'ubuntu/xenial64'
      server_log.vm.network "private_network", type: "dhcp"
      server_log.vm.hostname = "server-log"
      server_log.vm.synced_folder '.', '/vagrant', disabled: true
  end

  config.vm.define "client-log" do |client_log|
      client_log.vm.box = 'ubuntu/xenial64'
      client_log.vm.network "private_network", type: "dhcp"
      client_log.vm.hostname = "client-log"
      client_log.vm.synced_folder '.', '/vagrant', disabled: true
  end
end
