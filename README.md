# 🚀 Sacas Network Node Setup

---

## 📌 Prerequisites
Ensure you have the following installed:
- Ubuntu 20.04 / 22.04
- `curl`, `git`, `make`, `gcc`, and `jq` installed:
  ```bash
  sudo apt update && sudo apt install curl git make gcc jq -y
  ```

---

## 🔹 1. Install Go (Golang)
```bash
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.24.1.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
```
Verify the installation:
```bash
go version
```

To make Go available in future sessions, add it to your shell profile:
```bash
echo 'export PATH=/usr/local/go/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

---

## 🔹 2. Clone and Build `sacasd`
```bash
git clone https://github.com/sacasnetwork/sacas.git
cd sacas
make install
```

Verify installation:
```bash
sacasd version
```

Add `sacasd` to your system path:
```bash
export PATH=$HOME/go/bin:$PATH
echo 'export PATH=$HOME/go/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

---

## 🔹 3. Initialize Node
```bash
sacasd init "Your Node Name" --chain-id sac_1317-1
```

This creates the `.sacasd` directory with the default configuration.

---

## 🔹 4. Generate Keys & Genesis Setup
Create a new key:
```bash
sacasd keys add validator
```

Add the account to `genesis.json`:
```bash
sacasd add-genesis-account $(sacasd keys show validator -a) 100000000000000000000asacas
```

Create the genesis transaction:
```bash
sacasd gentx validator 100000000000000000000asacas --chain-id sac_1317-1
```

Collect the `gentx`:
```bash
sacasd collect-gentxs
```

---

## 🔹 5. Fix Missing Validator State File (If Needed)
If you encounter issues with `priv_validator_state.json`, create it manually:
```bash
echo '{"height":"0","round":0,"step":0}' > ~/.sacasd/data/priv_validator_state.json
```

Get the validator address:
```bash
sacasd debug addr $(sacasd keys show validator -a)
```

---

## 🔹 6. Create a Systemd Service for `sacasd`

Create a new systemd service file:
```bash
nano /etc/systemd/system/sacasd.service
```

Paste the following configuration:
```ini
[Unit]
Description=Sacas Node
After=network.target

[Service]
User=root
ExecStart=/root/go/bin/sacasd start --chain-id sac_1317-1
Restart=always
RestartSec=3
LimitNOFILE=1000000

[Install]
WantedBy=multi-user.target
```

Save the file (`CTRL + X`, then `Y` and `Enter`).

Reload systemd and enable the service:
```bash
systemctl daemon-reload
systemctl enable sacasd
systemctl start sacasd
```

Check service status:
```bash
systemctl status sacasd
```

To view logs in real-time:
```bash
journalctl -u sacasd -f -o cat
```

---

## 🎯 **Node is now running!** 🚀

Your node is now part of the `Sacas Network` and will restart automatically on reboot or failure.

For troubleshooting, use:
```bash
sacasd status
sacasd query bank balances $(sacasd keys show validator -a)
```

