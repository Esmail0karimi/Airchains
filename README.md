# Airchains

# WARNING: To avoid losing your points in case of any error, keep the keys and priv keys given during the installation stages.

We install standard updates and requirements.

# Equipment

Minimum: 2 vCPU 4 RAM
Önerilen: 4vCPU 8 RAM

# Setup

# update
       apt update && apt upgrade -y
       sudo apt install -y curl git jq lz4 build-essential cmake perl automake autoconf libtool wget libssl-dev

# go setup
     sudo rm -rf /usr/local/go
     curl -L https://go.dev/dl/go1.22.3.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
     echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
     source .bash_profile

# We withdraw the necessary repos
     git clone https://github.com/airchains-network/evm-station.git
     git clone https://github.com/airchains-network/tracks.git

# We are starting to set up our evmos network, this is our own network running locally.
        cd evm-station
        go mod tidy

# We complete the installation with this command.

        /bin/bash ./scripts/local-setup.sh


# RPC will be needed in the next stages, let's set it up.
  # The RPC section at the bottom will be like this.

         nano ~/.evmosd/config/app.toml

  # So you learned how to make cosmos rpcs public.
   We create an env for the system file to function properly.  
        
        nano ~/.rollup-env

   # We enter the necessary variables into it.
   There is nothing to change in this code block..
   
     CHAINID="stationevm_9000-1"
     MONIKER="localtestnet"
     KEYRING="test"
     KEYALGO="eth_secp256k1"
     LOGLEVEL="info"
     HOMEDIR="$HOME/.evmosd"
     TRACE=""
     BASEFEE=1000000000
     CONFIG=$HOMEDIR/config/config.toml
     APP_TOML=$HOMEDIR/config/app.toml
     GENESIS=$HOMEDIR/config/genesis.json
     TMP_GENESIS=$HOMEDIR/config/tmp_genesis.json
        VAL_KEY="mykey"


# We write the service file. If you are using user, change the root section accordingly.
You can copy paste the entire block with one command, my dears.


    sudo tee /etc/systemd/system/rolld.service > /dev/null << EOF
    [Unit]
    Description=ZK
    After=network.target
 
    [Service]
    User=root
    EnvironmentFile=/root/.rollup-env
    ExecStart=/root/evm-station/build/station-evm start --metrics "" --log_level info --json-rpc.api eth,txpool,personal,net,debug,web3 --chain-id stationevm_9000-1
    Restart=always
    RestartSec=3

    [Install]
    WantedBy=multi-user.target
    EOF


# We update and start the services.
      sudo systemctl daemon-reload
      sudo systemctl enable rolld
      sudo systemctl start rolld
      sudo journalctl -u rolld -f --no-hostname -o cat

# This command will give us a private key, we keep it.

    /bin/bash ./scripts/local-keys.sh

# We will use eigenlayer as the DA layer. A key is required for this, we download the binary and allow it to run. There are also celestia and avail installations in the official documentation, you can look at them too. You can also use mock DA (they will allow you to earn points with mock for a while). Currently, DA cannot be changed later on the testnet, they said they will make this possible with the update.

The reason I chose EigenDA is because it is the easiest Celestia and Eigen (the token is also easy to find), we know Celestia by heart - let it be Eigen this time.

    cd $HOME
    wget https://github.com/airchains-network/tracks/releases/download/v0.0.2/eigenlayer
    mkdir -p $HOME/go/bin
    chmod +x $HOME/eigenlayer
    mv $HOME/eigenlayer $HOME/go/bin

# Change `WALLANAME` and keep the ECDSA Private Key given to you in the output.
# Close it with Ctrl+c, enter and note the other `public hex` given, you will need it.
# Add 0.5 eth to the given 0x evm address on the holesky network, just in case.

     eigenlayer operator keys create --key-type ecdsa CUZDANADI

# Now we move on to the track and station part.

    cd $HOME
    cd tracks
    go mod tidy

# While in the tracks folder, we start the following code. PUBLICHEX will be the public key we just received.
You can change the MONIKER (validator name) as per your preference.

     go run cmd/main.go init --daRpc "disperser-holesky.eigenda.xyz" --daKey "PUBLICHEX" --daType "eigen" --moniker "MONIKER" --stationRpc "http://127.0.0.1:8545" --stationAPI 
     "http://127.0.0.1:8545" --stationType "evm"

# Now we create the tracker address. Replace from TRACKERCUZ.
Back up the output, buy tokens from the switchyard faucet channel on discord with an air prefix wallet.

      go run cmd/main.go keys junction --accountName TRACKERCUZDAN --accountPath $HOME/.tracks/junction-accounts/keys

# Then we run the prover.

     go run cmd/main.go prover v1EVM

# Now we need node id, we get it from here.
You can search for node id with ctrl w, go to the bottom and go a little higher

    nano ~/.tracks/config/sequencer.toml

# In the code below
The name you wrote above from TRACKERCUZ
TRACKERCUZDAN-ADDRESS also air wallet
your ip ip address
NODEID will be the node id we obtained from sequencer.toml.

     go run cmd/main.go create-station --accountName TRACKERCUZDAN --accountPath $HOME/.tracks/junction-accounts/keys --jsonRPC "https://airchains-testnet-rpc.cosmonautstakes.com/" -- 
     info "EVM Track" --tracks TRACKERCUZDAN-ADRESI --bootstrapNode "/ip4/IP/tcp/2300/p2p/NODEID"


# We've set up the station, now let's run it with the service.
Those who do not want to run the service can open a screen and run the go run cmd/main.go start command in the tracks folder.

   sudo tee /etc/systemd/system/stationd.service > /dev/null << EOF
   [Unit]
   Description=station track service
   After=network-online.target
   [Service]
   User=root
   WorkingDirectory=/root/tracks/
   ExecStart=/usr/local/go/bin/go run cmd/main.go start
   Restart=always
   RestartSec=3
   LimitNOFILE=65535
   [Install]
   WantedBy=multi-user.target
   EOF

# and 

       sudo systemctl daemon-reload
       sudo systemctl enable stationd
       sudo systemctl restart stationd
       sudo journalctl -u stationd -f --no-hostname -o cat

# Is the installation ok?

That's all the installation process. But you don't earn points right now. We import the mnemonics of your Tracker wallet into leap wallet and click https://points.airchains.io/ connect. You can see your station and points on the Dashboard. Since we have not made a tx yet, 100 points pending will appear. The reason for this is that you need to take out pods to earn points. You can think of it as a package consisting of pod 25tx. Every 25tx will extract 1 pod and you will earn 5 points from these transactions. The 100 points in the first installation will be active after the first pod.
For this, we do the following. First, we got a priv key with the bin/bash ./scripts/local-keys.sh command and set the rpc. We import this priv key to Metamaska, in the add network section

rpc http://IP:8545

id 9000

ticker tEVMOS

# We enter and okay.

From here, it's up to you whether you deploy a contract or a manual TX.

Those who get an rpc error during the track process should try rollback. Sometimes the problem is solved with 1, sometimes 3 rollback operations. Run the go run cmd/main.go rollback command as many times as you want to rollback, waiting for the output each time.

   systemctl stop stationd
   cd tracks
   git pull
   go run cmd/main.go rollback
   sudo systemctl restart stationd
   sudo journalctl -u stationd -f --no-hostname -o cat

#  Come on, goodbye
