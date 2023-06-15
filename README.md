```sh
wget https://raw.githubusercontent.com/stakingcabin/celestia-task/main/consensus.py
```

# consensus
A Simple tool for check validator sign status for recent 100 blocks and the state of the consensus

<details>
  <summary>Quick start:</summary>

```sh
cd && git clone https://github.com/stakingcabin/celestia-task.git && cd celestia-task
# show usage
python3 consensus.py --help
# show local validator sign status for recent 100 blocks
python3 consensus.py sign-check
# show validators consensus staus 
python3 consensus.py

```  
</details>

<details>
  <summary>Requirements:</summary>
  
  *  Ubuntu 20.04 
  *  python3.8 
  *  pip3 
  *  For the correct work of the application you should configure RPC 127.0.0.1:26657 and REST 127.0.0.1:1317 endpoints.  
  
  
</details>

<details>
  <summary>Installing:</summary>
  
  #### Technically, the installation itself is cloning the repo on your validator

```sh
$ cd && git clone https://github.com/stakingcabin/celestia-task.git && cd celestia-task
```  
you can run the app by following:
  
  ```$ python3 consensus.py sign-check```


</details>



![Example](https://github.com/stakingcabin/celestia-task/blob/main/screenshots/SignStatus.png?raw=true "EX")
![Example](https://github.com/stakingcabin/celestia-task/blob/main/screenshots/SignStatusWithFailed.png?raw=true "EX")

  ```$ python3 consensus.py```
![Example](https://github.com/stakingcabin/celestia-task/blob/main/screenshots/consensus.png?raw=true "EX")

Thanks https://github.com/Northa/consensus.git, Part code copy from that repo
