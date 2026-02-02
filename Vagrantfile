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
    apt-get install -y docker.io git unzip

    usermod -aG docker vagrant

    chown -R vagrant:vagrant /agent-workspace
    chown -R vagrant:vagrant /openclaw-workspace

    curl -fsSL https://tailscale.com/install.sh | sh
    tailscale up --ssh --operator=vagrant --authkey #{ENV['TAILSCALE_AUTHKEY']}
  SHELL

  config.vm.provision "shell", inline: <<-SHELL, privileged: false
    whoami

    # Download and install nvm:
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
    # in lieu of restarting the shell
    \. "$HOME/.nvm/nvm.sh"
    # Download and install Node.js:
    nvm install 24
    # Verify the Node.js version:
    node -v # Should print "v24.13.0".
    # Verify npm version:
    npm -v # Should print "11.6.2".

    curl -fsSL https://claude.ai/install.sh | bash
    curl -fsSL https://ampcode.com/install.sh | bash
    npm i -g @openai/codex @google/gemini-cli @mariozechner/pi-coding-agent

    git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
    ~/.fzf/install --all
  SHELL

  config.trigger.before :destroy do |trigger|
    trigger.run_remote = {inline: "tailscale logout"}
    trigger.on_error = :continue
  end
end
