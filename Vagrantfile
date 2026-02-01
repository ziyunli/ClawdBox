vm_name = File.basename(Dir.getwd)

# Load .env file manually (replaces vagrant-env plugin)
if File.exist?('.env')
  File.readlines('.env').each do |line|
    line = line.strip
    next if line.empty? || line.start_with?('#')
    key, value = line.split('=', 2)
    ENV[key] = value if key && value
  end
end

Vagrant.configure("2") do |config|
  config.vm.box = ENV['VM_BOX']

  config.vm.network "forwarded_port", guest: 3000, host: 3000, auto_correct: true
  config.vm.network "forwarded_port", guest: 18789, host: 18789
  config.vm.synced_folder "./agent-workspace", "/agent-workspace", type: "virtualbox"
  config.vm.synced_folder "./openclaw-workspace", "/openclaw-workspace", type: "virtualbox"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "4096"
    vb.cpus = 2
    vb.gui = false
    vb.name = vm_name
    vb.customize ["modifyvm", :id, "--audio", "none"]
    vb.customize ["modifyvm", :id, "--usb", "off"]
  end

  config.vm.provision "shell", inline: <<-SHELL
    export DEBIAN_FRONTEND=noninteractive

    apt-get update
    apt-get install -y docker.io nodejs npm git unzip

    curl -fsSL https://claude.ai/install.sh | bash
    curl -fsSL https://ampcode.com/install.sh | bash
    npm i -g @openai/codex @google/gemini-cli @mariozechner/pi-coding-agent

    usermod -aG docker vagrant

    chown -R vagrant:vagrant /agent-workspace
    chown -R vagrant:vagrant /openclaw-workspace
  SHELL

  config.vm.provision "tailscale-install", type: "shell" do |s|
    s.inline = "curl -fsSL https://tailscale.com/install.sh | sh"
  end

  config.vm.provision "tailscale-up", type: "shell" do |s|
    s.inline = "tailscale up --ssh --operator=vagrant --authkey #{ENV['TAILSCALE_AUTHKEY']}"
  end

  # When Tailscale is installed in the VM, it configures MagicDNS (100.100.100.100)
  # as the DNS resolver. This can break external DNS resolution (e.g., api.telegram.org)
  # if your Tailscale network doesn't have global nameservers configured,
  # or if the configuration doesn't propagate correctly.
  # To revert:
  # sudo resolvectl revert eth0
  # sudo systemctl restart systemd-resolved
  # dig @100.100.100.100 api.telegram.org
  config.vm.provision "fix-dns", type: "shell" do |s|
    s.inline = "resolvectl dns eth0 1.1.1.1 8.8.8.8"
  end
end
