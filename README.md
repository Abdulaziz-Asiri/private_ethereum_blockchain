# private_ethereum_blockchain
# Private Ethereum Blockchain Setup on AWS

This guide explains how to set up a private Ethereum blockchain on AWS using `geth` (Go Ethereum). Follow these steps to configure the blockchain nodes and connect them securely.

---

## **Prerequisites**

### **AWS Requirements**
- Two AWS EC2 instances (Ubuntu 20.04 or later recommended).
- Open ports for Ethereum communication:
  - TCP/UDP `30303`: For peer-to-peer communication.
  - TCP `8545`: For JSON-RPC (optional).
- Security Group allowing required inbound and outbound traffic.

### **Local Setup Requirements**
- SSH access to your EC2 instances.
- Basic knowledge of Linux commands.
- Installed tools:
  - `geth` (Go Ethereum)
  - `openssl` (for generating a genesis file and keys)

---

## **Step 1: Install Geth on AWS EC2 Instances**

1. SSH into each EC2 instance:
   ```bash
   ssh -i your-key.pem ubuntu@<INSTANCE_IP>
   ```

2. Update and install dependencies:
   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo apt install -y software-properties-common
   sudo add-apt-repository -y ppa:ethereum/ethereum
   sudo apt update
   sudo apt install -y ethereum
   ```

3. Verify installation:
   ```bash
   geth version
   ```

---

## **Step 2: Create a Genesis Block**

1. Creating working directory:
    ```bash 
    mkdir ~/private-eth
    cd ~/private-eth
    ```
2. Create account:
    ```bash
    geth --datadir=./datadir account new
    ```

2. On one of the EC2 instances, create a `genesis.json` file:
   ```bash
   nano genesis.json
   ```

3. Paste the following configuration into the file:
   ```json
   
        {
        "config": {
            "chainId": 12345,
            "homesteadBlock": 0,
            "eip150Block": 0,
            "eip155Block": 0,
            "eip158Block": 0,
            "byzantiumBlock": 0,
            "constantinopleBlock": 0,
            "petersburgBlock": 0,
            "istanbulBlock": 0,
            "berlinBlock": 0,
            "clique": {
            "period": 10,
            "epoch": 3000000
            }
        },
        "difficulty": "1",
        "gasLimit": "8000000",
        "extradata": "0x0000000000000000000000000000000000000000000000000000000000000000<ACCOUNT ADDRESS>0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"
        ,
        "alloc": {
            "<ACCOUNT ADDRESS>": { "balance": "50000000" }
        }
    }

   ```

4. Initialize the genesis block:
   ```bash
   geth init genesis.json
   ```

---

## **Step 3: Start Ethereum Nodes**

1. Start the first node:
   ```bash
   geth --networkid 1234 --nodiscover --http --http.addr "0.0.0.0" --http.port 8545 --datadir node1 --port 30303
   ```

2. On the second instance, start the second node:
   ```bash
   geth --networkid 1234 --nodiscover --http --http.addr "0.0.0.0" --http.port 8546 --datadir node2 --port 30304
   ```

---

## **Step 4: Connect the Nodes**

1. On the first node, get its enode address:
   ```bash
   geth attach http://127.0.0.1:8545
   > admin.nodeInfo.enode
   ```

   The output will look like this:
   ```
   "enode://<NODE_ID>@<INSTANCE_1_IP>:30303"
   ```

2. On the second node, attach to the console:
   ```bash
   geth attach http://127.0.0.1:8546
   > admin.addPeer("enode://<NODE_ID>@<INSTANCE_1_IP>:30303")
   ```

3. Verify the connection from the first node:
   ```bash
   geth attach http://127.0.0.1:8545
   > admin.peers
   ```
   If successful, the output will list the second node.

---

## **Step 5: Optional - Enable Mining**

1. Start mining on the first node:
   ```bash
   geth --mine --miner.threads 1 --datadir node1
   ```

2. To unlock an account and send transactions, use the console:
   ```bash
   geth attach http://127.0.0.1:8545
   > personal.newAccount("password")
   > miner.start()
   ```

---

## **Step 6: Security and Maintenance**

- Ensure security group rules allow only necessary traffic.
- Use a firewall (e.g., UFW) to restrict access further.
- Regularly monitor logs:
  ```bash
  tail -f /var/log/geth.log
  ```

---

## **Troubleshooting**

- If nodes fail to connect:
  - Verify security group rules.
  - Ensure both nodes use the same `genesis.json` and `networkid`.
- Check logs for detailed errors.
  ```bash
  cat /node1/geth/logs
  ```

---

## **References**
- [Geth Documentation](https://geth.ethereum.org/docs/)
- [AWS EC2 Setup](https://aws.amazon.com/ec2/)

---

Now you have a private Ethereum blockchain running on AWS! Modify it to suit your specific requirements.
