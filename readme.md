#### SUI - [Air drop](https://twitter.com/ardizor/thread/1642874737390133249)
- It's a scam, no air drops. The instructions are how to setup a dev node.
- Most people out there are abusing the faucet.

##### Instance Resouces:

- EC2 Instance (ap-southeast-1)  
<img width="316" alt="Screenshot 2023-04-05 at 23 34 18" src="https://user-images.githubusercontent.com/69737492/230226705-9462ebf8-bf05-4470-9b32-d1cde102d7fa.png">
<img width="191" alt="Screenshot 2023-04-05 at 23 34 27" src="https://user-images.githubusercontent.com/69737492/230226723-b9f2800a-45d7-4f97-b675-e30fe6990605.png">
<img width="519" alt="Screenshot 2023-04-05 at 23 34 44" src="https://user-images.githubusercontent.com/69737492/230226732-bf875715-f42a-46e1-b4d8-33af62bdafc3.png">


Connection:

```shell
chmod 400 SUI.pem

ssh -i "SUI.pem" ubuntu@ec2-3-1-102-163.ap-southeast-1.compute.amazonaws.com

sudo su # always run as sudo user #root
```

Prep:

```shell
sudo apt update && sudo apt upgrade -y

sudo apt install wget jq git libclang-dev cmake -y

curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh # Type "1" and click enter.

source "$HOME/.cargo/env"

rustc --version
```

Installation:

```shell
mkdir $HOME/.sui

git clone https://github.com/MystenLabs/sui.git --branch devnet

cd sui

cargo build --release
```

Debug Code:

```shell
version=`wget -qO- https://api.github.com/repos/SecorD0/Sui/releases/latest | jq -r ".tag_name"`;\

wget -qO- "https://github.com/SecorD0/Sui/releases/download/${version}/sui-linux-amd64-${version}.tar.gz" | tar -C /usr/bin/ -xzf -
```

<details>
<summary>
If the last action failed and filed already exists:
</summary>

```shell
rm -rf sui sui-faucet sui-node sui-indexer
```
</details>

Be extract careful when executing...

```shell
mv $HOME/sui/target/release/{sui,sui-node,sui-faucet} /usr/bin/

cd && wget -qO $HOME/.sui/genesis.blob https://github.com/MystenLabs/sui-genesis/raw/main/devnet/genesis.blob

cp $HOME/sui/crates/sui-config/data/fullnode-template.yaml  $HOME/.sui/fullnode.yaml

sed -i -e "s%db-path:.*%db-path: \"$HOME/.sui/db\"%; "\
"s%metrics-address:.*%metrics-address: \"0.0.0.0:9184\"%; "\
"s%json-rpc-address:.*%json-rpc-address: \"0.0.0.0:9000\"%; "\ 
"s%genesis-file-location:.*%genesis-file-location: \"$HOME/.sui/genesis.blob\"%; " $HOME/.sui/fullnode.yaml

sudo ufw allow 9000 && sudo ufw allow 9184
```

```shell
printf "[Unit]
Description=Sui node
After=network-online.target

[Service]
User=$USER
ExecStart=`which sui-node` --config-path $HOME/.sui/fullnode.yaml
Restart=on-failure RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/suid.service
```

```shell
sudo systemctl daemon-reload
sudo systemctl enable suid
sudo systemctl restart suid && journalctl -u suid -f
```

Done!, Visit:

https://node.sui.zvalid.com

#### How to update the node:

```shell
systemctl stop suid

rm -rf $HOME/.sui/db

wget -qO $HOME/.sui/genesis.blob https://github.com/MystenLabs/sui-genesis/raw/main/devnet/genesis.blob…

cd $HOME/sui

git fetch upstream

cargo build --release

mv $HOME/sui/target/release/{sui,sui-node,sui-faucet} /usr/bin/

sui -V

systemctl restart suid
```
