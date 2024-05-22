# Initia-node
Hardware requirements
- Memory: 16 GB RAM
- CPU: 4 cores
- Disk: 1 TB SSD
- Bandwidth: 1 Gbps
- Linux amd64 arm64 (Ubuntu LTS release)


Installation guide
1. Pre-Install  sudo apt install tmux git -y && tmux new -s initia
2. Install    curl -o install_initia.sh https://raw.githubusercontent.com/solotop999/validator_initia/main/install_initia.sh
chmod +x install_initia.sh

.Change 'YOUR_NODE_NAME' to your name:  ./install_initia.sh YOUR_NODE_NAME


.Following script insctruction:
Backup your mnemonic and public key.
Go to: https://faucet.testnet.initia.xyz/ and claim faucet
Press Enter to continute...
![image](https://github.com/tonycafe007/Initia-node/assets/128784616/3e469899-3036-4639-8fbc-59e9d4bc1aac)
Waiting to it sync full block. (few hours)
Block left must be 0 and it will auto run next step
You can turn off terminal and check status later

.check logs: journalctl -t initiad -f -o cat

.Check Blocks left: 
  local_height=$(/root/go/bin/initiad status | jq -r .sync_info.latest_block_height)
  network_height=$(curl -s https://rpc-initia-testnet.trusted-point.com/status | jq -r .result.sync_info.latest_block_height)
  blocks_left=$((network_height - local_height))
  echo ""
  echo "Your node height: $local_height"
  echo "Network height: $network_height"
  echo " => Blocks left: $blocks_left <="

 . Access to tmux:  tmux attach -t initia

  DONE ALL.
  If your blocks_left no decresed
Download latest snapshot
1.Stop the service and reset the data
sudo systemctl stop initiad.service
cp $HOME/.initia/data/priv_validator_state.json $HOME/.initia/priv_validator_state.json.backup
rm -rf $HOME/.initia/data

2. Download latest snapshot
curl -L https://snapshots.kzvn.xyz/initia/initiation-1_latest.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.initia
mv $HOME/.initia/priv_validator_state.json.backup $HOME/.initia/data/priv_validator_state.json

3.Restart the service and check the log
sudo systemctl restart initiad.service && sudo journalctl -u initiad.service -f --no-hostname -o cat

.Change peer. Choose 1 or 2
1. NON CONTABO PEERS (225)

PEERS=$(curl -s --max-time 3 --retry 2 --retry-connrefused "https://rpc-initia-testnet.trusted-point.com/non_contabo_peers.txt")

2. HETZNER PEERS (45)
   PEERS=$(curl -s --max-time 3 --retry 2 --retry-connrefused "https://rpc-initia-testnet.trusted-point.com/hetzner_peers.txt")

.Then
if [ -z "$PEERS" ]; then
    echo "No peers were retrieved from the URL."
else
    echo -e "\nPEERS: "$PEERS""
    sed -i "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" "$HOME/.initia/config/config.toml"
    echo -e "\nConfiguration file updated successfully.\n"
fi

.Stop the node:  sudo systemctl stop initiad
.Delete addrbook.json: sudo rm $HOME/.initia/config/addrbook.json
.Start the node: sudo systemctl restart initiad
.Check logs: sudo journalctl -u initiad -f -o cat --no-hostname
.Check sync: initiad status | jq -r .sync_info

If the blocks go decresed, you will be ok. Wait the Blocks_left = 0.





