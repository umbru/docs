# Masternodes

## Setup

Setting up a masternode requires a basic understanding of Linux and blockchain technology, as well as an ability to follow instructions closely. It also requires regular maintenance and careful security, particularly if you are not storing your Umbru on a hardware wallet. There are some decisions to be made along the way, and optional extra steps to take for increased security.

### Requirements

* 5000 Umbru
* A Umbru wallet \(Umbru Core or hardware wallet\)
* A Linux server, preferably a Virtual Private Server \(VPS\)

### Setting up VPS

Login to your VPS and we will update the system from the Ubuntu package repository:

```text
apt update
apt upgrade
```

 The system will show a list of upgradable packages. Press **Y** and **Enter** to install the packages. We will now install a firewall \(and some other packages we will use later\), add swap memory and reboot the server to apply any necessary kernel updates, and then login to our newly secured environment as the new user:

```text
apt install ufw python virtualenv git unzip pv
```

\(press **Y** and **Enter** to confirm\)

```text
ufw allow ssh/tcp
ufw limit ssh/tcp
ufw allow 12353/tcp
ufw logging on
ufw enable
```

\(press **Y** and **Enter** to confirm\)

```text
fallocate -l 4G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
nano /etc/fstab
```

Add the following line at the end of the file \(press tab to separate each word/number\), then press **Ctrl + X** to close the editor, then **Y** and **Enter** save the file.

```text
/swapfile none swap sw 0 0
```

### Sending collateral

A Umbru address with a single unspent transaction output \(UTXO\) of exactly 5000 UMBRU is required to operate a masternode. Once it has been sent, various keys regarding the transaction must be extracted for later entry in a configuration file and registration transaction as proof to write the configuration to the blockchain so the masternode can be included in the deterministic list. A masternode can be registered from the official Umbru Core wallet.

Click **Tools &gt; Debug console** to open the console. Type the following command into the console to generate a new Umbru address for the collateral:

```text
getnewaddress
UXvkAbdfwCXxGjaPDmAmw5uNqGHmEoFiNS
```

Take note of the collateral address, since we will need it later. The next step is to secure your wallet \(if you have not already done so\). First, encrypt the wallet by selecting **Settings &gt; Encrypt wallet**. You should use a strong, new password that you have never used somewhere else. Take note of your password and store it somewhere safe or you will be permanently locked out of your wallet and lose access to your funds. Next, back up your wallet file by selecting **File &gt; Backup Wallet**. Save the file to a secure location physically separate to your computer, since this will be the only way you can access our funds if anything happens to your computer. 

Now send exactly 5000 UMBRU in a single transaction to the new address you generated in the previous step. This may be sent from another wallet, or from funds already held in your current wallet. Once the transaction is complete, view the transaction in a [blockchain explorer](https://explorer.umbru.io) by searching for the address. You will need 15 confirmations before you can register the masternode, but you can continue with the next step at this point already: generating your masternode operator key.

### Installing Umbru Core on VPS

Go to your VPS terminal window and enter the following command, to get the latest version of Umbru Core by right clicking or pressing **Ctrl + V**:

```text
wget https://github.com/umbru/umbru-core/releases/download/v0.14.0.2/umbrucore-0.14.0.2-x86_64-linux-gnu.tar.gz
```

Create a working directory for Umbru, extract the compressed archive and copy the necessary files to the directory by using the following commands:

```text
mkdir ~/.umbru
tar xfv umbrucore-0.14.0.2-x86_64-linux-gnu.tar.gz
cp -f umbrud ~/.umbru/
cp -f umbru-cli ~/.umbru/
```

Create a configuration file using the following command:

```text
nano ~/.umbru/umbru.conf
```

An editor window will appear. We now need to create a configuration file specifying several variables. Copy and paste the following text to get started, then replace the variables specific to your configuration as follows:

```text
#----
rpcuser=XXXXXXXXXXXXX
rpcpassword=XXXXXXXXXXXXXXXXXXXXXXXXXXXX
rpcallowip=127.0.0.1
#----
listen=1
server=1
daemon=1
maxconnections=64
#----
#masternode=1
#masternodeblsprivkey=
externalip=XXX.XXX.XXX.XXX
#----
```

Replace the fields marked with `XXXXXXX` as follows:

* `rpcuser`: enter any string of numbers or letters, no special characters allowed
* `rpcpassword`: enter any string of numbers or letters, no special characters allowed
* `externalip`: this is the IP address of your VPS

Leave the `masternode` and `masternodeblsprivkey` fields commented out for now. 

Press **Ctrl + X** to close the editor and **Y** and **Enter** save the file. You can now start running Umbru on the masternode to begin synchronization with the blockchain:

```text
~/.umbru/umbrud
```

You will see a message reading **Umbru Core server starting**. We will now install Sentinel, a piece of software which operates as a watchdog to communicate to the network that your node is working properly:

```text
cd ~/.umbru
git clone https://github.com/umbru/sentinel.git
cd sentinel
virtualenv venv
venv/bin/pip install -r requirements.txt
venv/bin/python bin/sentinel.py
```

You will see a message reading **umbrud not synced with network! Awaiting full sync before running Sentinel.** Add umbrud and sentinel to crontab to make sure it runs every minute to check on your masternode:

```text
crontab -e
```

Choose nano as your editor and enter the following lines at the end of the file:

```text
* * * * * cd ~/.umbru/sentinel && ./venv/bin/python bin/sentinel.py 2>&1 >> sentinel-cron.log
* * * * * pidof umbrud || ~/.umbru/umbrud
```

Press enter to make sure there is a blank line at the end of the file, then press **Ctrl + X** to close the editor and **Y** and **Enter** save the file. We now need to wait for 15 confirmations of the collateral transaction to complete, and wait for the blockchain to finish synchronizing on the masternode. You can use the following commands to monitor progress:

```text
~/.umbru/umbru-cli mnsync status
```

When synchronization is complete, you should see the following response:

```text
{
  "AssetID": 999,
  "AssetName": "MASTERNODE_SYNC_FINISHED",
  "AssetStartTime": 1566039979,
  "Attempt": 0,
  "IsBlockchainSynced": true,
  "IsSynced": true,
  "IsFailed": false
}
```

Continue with the next step to construct the ProTx transaction required to enable your masternode.

### Registering Masternode

**Identify the funding transaction**

If you used an address in Umbru Core wallet for your collateral transaction, you now need to find the txid of the transaction. Click **Tools &gt; Debug console** and enter the following command:

```text
masternode outputs
```

This should return a string of characters similar to the following:

```text
{
"16347a28f4e5edf39f4dceac60e2327931a25fdee1fb4b94b63eeacf0d5879e3" : "1",
}
```

The first long string is your `collateralHash`, while the last number is the `collateralIndex`.

**Generate a BLS key pair**

A public/private BLS key pair is required to operate a masternode. The private key is specified on the masternode itself, and allows it to be included in the deterministic masternode list once a provider registration transaction with the corresponding public key has been created.

If you are using a hosting service, they may provide you with their public key, and you can skip this step. If you are hosting your own masternode or have agreed to provide your host with the BLS private key, generate a BLS public/private keypair in Umbru Core by clicking **Tools &gt; Debug console** and entering the following command:

```text
bls generate

{
  "secret": "6d7280762db6e7322ccce9bed2c9664d7fc186e9f21cf68ebb22f29cafd94eed",
  "public": "93fc08a30a02880d2c85275d735c223fbb64c070b133bb214f69cbe5db70d3f90285bb47cd48d58c24796e1156d5aeaf"
}
```

**These keys are NOT stored by the wallet and must be kept secure.**

**Add the private key to your masternode configuration**

The public key will be used in following steps. The private key must be entered in the `umbru.conf` file on the masternode. This allows the masternode to watch the blockchain for relevant ProTx transactions, and will cause it to start serving as a masternode when the signed ProRegTx is broadcast by the owner \(final step below\). Log in to your masternode using `ssh` or PuTTY and edit the configuration file as follows:

```text
nano ~/.umbru/umbru.conf
```

The editor appears with the existing masternode configuration. Add or uncomment these lines in the file, replacing the key with your BLS private key generated above:

```text
masternode=1
masternodeblsprivkey=6d7280762db6e7322ccce9bed2c9664d7fc186e9f21cf68ebb22f29cafd94eed
```

Press enter to make sure there is a blank line at the end of the file, then press **Ctrl + X** to close the editor and **Y** and **Enter** save the file. We now need to restart the masternode for this change to take effect. Enter the following commands, waiting a few seconds in between to give Umbru Core time to shut down:

```text
~/.umbru/umbru-cli stop
sleep 15
~/.umbru/umbrud
```

We will now prepare the transaction used to register the masternode on the network.

**Prepare a ProRegTx transaction**

A pair of BLS keys for the operator were already generated above, and the private key was entered on the masternode. The public key is used in this transaction as the `operatorPubKey`.

First, we need to get a new, unused address from the wallet to serve as the **owner key address** \(`ownerKeyAddr`\). This is not the same as the collateral address holding 5000 UMBRU. Generate a new address as follows:

```text
getnewaddress

UdohSjHe3t9Z6DUYFJsTYTZwikQMRtZN3h
```

This address can also be used as the **voting key address** \(`votingKeyAddr`\). Alternatively, you can specify an address provided to you by your chosen voting delegate, or simply generate a new voting key address as follows:

```text
getnewaddress

Ua5XQGvSoTE5M92J1oo7vGdPnxc5w3LZfr
```

Then either generate or choose an existing address to receive the **owner’s masternode payouts** \(`payoutAddress`\). It is also possible to use an address external to the wallet:

```text
getnewaddress

UXuHYc1C8b9RcGNw6iRkRTqKPMfXFWqZbK
```

You can also optionally generate and fund another address as the **transaction fee source**\(`feeSourceAddress`\). If you selected an external payout address, you must specify a fee source address. Either the payout address or fee source address must have enough balance to pay the transaction fee, or the final `register_submit` transaction will fail.

The private keys to the owner and fee source addresses must exist in the wallet submitting the transaction to the network. If your wallet is protected by a password, it must now be unlocked to perform the following commands. Unlock your wallet for 5 minutes:

```text
walletpassphrase yourSecretPassword 300
```

We will now prepare an unsigned ProRegTx special transaction using the `protx register_prepare`command. This command has the following syntax:

```text
protx register_prepare collateralHash collateralIndex ipAndPort ownerKeyAddr
  operatorPubKey votingKeyAddr operatorReward payoutAddress (feeSourceAddress)
```

Open a text editor such as notepad to prepare this command. Replace each argument to the command as follows:

* `collateralHash`: The txid of the 5000 UMBRU collateral funding transaction
* `collateralIndex`: The output index of the 5000 UMBRU funding transaction
* `ipAndPort`: Masternode IP address and port, in the format `x.x.x.x:yyyy`
* `ownerKeyAddr`: The new Umbru address generated above for the owner/voting address
* `operatorPubKey`: The BLS public key generated above \(or provided by your hosting service\)
* `votingKeyAddr`: The new Umbru address generated above, or the address of a delegate, used for proposal voting
* `operatorReward`: The percentage of the block reward allocated to the operator as payment
* `payoutAddress`: A new or existing Umbru address to receive the owner’s masternode rewards
* `feeSourceAddress`: An \(optional\) address used to fund ProTx fee. `payoutAddress` will be used if not specified.

Note that the operator is responsible for specifying their own reward address in a separate `update_service` transaction if you specify a non-zero `operatorReward`. The owner of the masternode collateral does not specify the operator’s payout address.Example \(remove line breaks if copying\):

```text
protx register_prepare
  6d7280762db6e7322ccce9bed2c9664d7fc186e9f21cf68ebb22f29cafd94eed
  1
  45.76.230.239:12353
  UXvkAbdfwCXxGjaPDmAmw5uNqGHmEoFiNS
  99f20ed1538e28259ff80044982372519a2e6e4cdedb01c96f8f22e755b2b3124fbeebdf6de3587189cf44b3c6e7670e
  UdohSjHe3t9Z6DUYFJsTYTZwikQMRtZN3h
  0
  Ua5XQGvSoTE5M92J1oo7vGdPnxc5w3LZfr
  UXuHYc1C8b9RcGNw6iRkRTqKPMfXFWqZbK
```

Output:

```text
{
  "tx": "030001000175c9d23c2710798ef0788e6a4d609460586a20e91a15f2097f56fc6e007c4f8e0000000000feffffff01a1949800000000001976a91434b09363474b14d02739a327fe76e6ea12deecad88ac00000000d1010000000000e379580dcfea3eb6944bfbe1de5fa2317932e260acce4d9ff3ede5f4287a34160100000000000000000000000000ffff2d4ce6ef4e1fd47babdb9092489c82426623299dde76b9c72d9799f20ed1538e28259ff80044982372519a2e6e4cdedb01c96f8f22e755b2b3124fbeebdf6de3587189cf44b3c6e7670ed1935246865dce1accce6c8691c8466bd67ebf1200001976a914fef33f56f709ba6b08d073932f925afedaa3700488acfdb281e134504145b5f8c7bd7b47fd241f3b7ea1f97ebf382249f601a0187f5300",
  "collateralAddress": "UXvkAbdfwCXxGjaPDmAmw5uNqGHmEoFiNS",
  "signMessage": "yjZVt49WsQd6XSrPVAUGXtJccxviH9ZQpN|0|yfgxFhqrdDG15ZWKJAN6dQvn6dZdgBPAip|yfRaZN8c3Erpqj9iKnmQ9QDBeUuRhWV3Mg|ad5f82257bd00a5a1cb5da1a44a6eb8899cf096d3748d68b8ea6d6b10046a28e"
}
```

Next we will use the `collateralAddress` and `signMessage` fields to sign the transaction, and the output of the `tx` field to submit the transaction.

**Sign the ProRegTx transaction**

We will now sign the content of the `signMessage` field using the private key for the collateral address as specified in `collateralAddress`. Note that no internet connection is required for this step, meaning that the wallet can remain disconnected from the internet in cold storage to sign the message. In this example we will again use Umbru Core, but it is equally possible to use the signing function of a hardware wallet. The command takes the following syntax:

```text
signmessage collateralAddress signMessage
```

Example:

```text
signmessage UXvkAbdfwCXxGjaPDmAmw5uNqGHmEoFiNS yjZVt49WsQd6XSrPVAUGXtJccxviH9ZQpN|0|yfgxFhqrdDG15ZWKJAN6dQvn6dZdgBPAip|yfRaZN8c3Erpqj9iKnmQ9QDBeUuRhWV3Mg|ad5f82257bd00a5a1cb5da1a44a6eb8899cf096d3748d68b8ea6d6b10046a28e
```

Output:

```text
II8JvEBMj6I3Ws8wqxh0bXVds6Ny+7h5HAQhqmd5r/0lWBCpsxMJHJT3KBcZ23oUZtsa6gjgISf+a8GzJg1BfEg=
```

**Submit the signed message**

We will now submit the ProRegTx special transaction to the blockchain to register the masternode. This command must be sent from a Umbru Core wallet holding a balance on either the `feeSourceAddress` or `payoutAddress`, since a standard transaction fee is involved. The command takes the following syntax:

```text
protx register_submit tx sig
```

Where:

* `tx`: The serialized transaction previously returned in the `tx` output field from the `protx register_prepare` command
* `sig`: The message signed with the collateral key from the `signmessage` command

Example:

```text
protx register_submit 030001000175c9d23c2710798ef0788e6a4d609460586a20e91a15f2097f56fc6e007c4f8e0000000000feffffff01a1949800000000001976a91434b09363474b14d02739a327fe76e6ea12deecad88ac00000000d1010000000000e379580dcfea3eb6944bfbe1de5fa2317932e260acce4d9ff3ede5f4287a34160100000000000000000000000000ffff2d4ce6ef4e1fd47babdb9092489c82426623299dde76b9c72d9799f20ed1538e28259ff80044982372519a2e6e4cdedb01c96f8f22e755b2b3124fbeebdf6de3587189cf44b3c6e7670ed1935246865dce1accce6c8691c8466bd67ebf1200001976a914fef33f56f709ba6b08d073932f925afedaa3700488acfdb281e134504145b5f8c7bd7b47fd241f3b7ea1f97ebf382249f601a0187f5300 II8JvEBMj6I3Ws8wqxh0bXVds6Ny+7h5HAQhqmd5r/0lWBCpsxMJHJT3KBcZ23oUZtsa6gjgISf+a8GzJg1BfEg=
```

Output:

```text
aba8c22f8992d78fd4ff0c94cb19a5c30e62e7587ee43d5285296a4e6e5af062
```

Your masternode is now registered and will appear on the Deterministic Masternode List after the transaction is mined to a block. You can view this list on the **Masternodes -&gt; Umbru Masternodes** tab of the Umbru Core wallet, or in the console using the command `protx list valid`, where the txid of the final `protx register_submit` transaction identifies your masternode.

