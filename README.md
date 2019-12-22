## ICO smart contract
Simple ICO smart contract for TON network

Detail information about this smart contract read in pdf files.

How to create, setup and upload smart contract to TON network:
#### 1. Compile lite-client as described [here](https://test.ton.org/README.txt)

Using *.fif scripts in folder "fift" do follow:

#### 2. Create wallet call it owner_wallet.

    $ <build-directory>/crypto/fift -I<source-directory>/crypto/fift/lib -s <this-directory>/fift/new-wallet.fif 0 owner_wallet

#### 3.Create second wallet and call it dest_wallet. 

    $ <build-directory>/crypto/fift -I<source-directory>/crypto/fift/lib -s <this-directory>/fift/new-wallet.fif 0 dest_wallet
  
Now we need to setup ICO smart contract. We know owner_wallet and dest_wallet addresses. Also we need find out date start and date end of ICO in unixtimestamp format. You can use this [site](https://www.unixtimestamp.com) to do it.
For example, we want to sell tokens in 4 stages. Each stage have some amount of tokens by some price.

stage | tokens | price 
------|--------|-------
  1   | 500000 | 1000 
  2   | 500000 | 1500 
  3   | 500000 | 2000 
  4   | 500000 | 2500

Next settings: softcup is 3000000000, date start - 1576972800, date end - 1579651200, Currency id - 239, bounty percent - 11,34% of sold tokens.

Let's setup ICO smart contract. 
Run in console follow command, replaced directories to your one.

    <build-directory>/crypto/fift -I<source-directory>/crypto/fift/lib -s <this-directory>/fift/setup-smc.fif 0 <owner_wallet> <dest_wallet> 3000000000 1576972800 1579651200 239 1134 500000 1000 500000 1500 500000 2000 500000 2500````

Pay attention, that stage info is last parameters and going after bounty percent which is multiplied by a hundred (1134).
This action will show smart contract addresses and will produce .boc file. Send 1 gram to init address of smart contract, then upload .boc file with lite-client (sendfile command).

Now smart contract ready to use.
You can send bounty stake to some wallet. Smart contract controlled from owner wallet, which you specified when setting up a smart contract. This process is simple sending small amount of grams to smart contract with control message. Unused grams will return back to your wallet. There are two control messages - `enroll_bounty` and `finalize_ico`. One more message current_stage may used by users to buy tokens by price current stage only (_read pdf documentation_). To form a transaction with this messages you need to use the `wallet.fif` script that you usually use to transfer funds, but attaching a boc file to it with the corresponding command as message body. Files current_stage.boc and finalize_ico.boc contain simple one-byte command. File _send-bounty-msg.boc_ should be created using script `send_bounty.fif`. This script accept such parameters as destination wallet and bounty stake amount. Messages enroll_bounty and `finalize_ico` accepted from `owner_wallet` only.

***

### Command examples:
#### 1. Setup ICO smart contract
    ~/full-node/crypto/fift -I ~/full-node-src/crypto/fift/lib -L Asm.fif -s <this-directory>/fift/setup-smc.fif 0 kQBP3h6wrnRgnMVFWT9p_wfCPsNLtEJHQ8cwLuCxPT2UbDq- kQBtkGvKJl918FQv2yezIEMgGDHZXE7P1hN0UhrhRhZmTuLT 3000000000 1577021088 1577021988 239 1134 500000 1000 500000 1500 500000 2000 500000 2500

Result:

_Smart contract address = 0:b18524e82c4c49d68b29047ae0b6aafbf055f0664d29a24f2ae579b163aa2531 
(Saving address to file ico.addr)
Non-bounceable address (for init): 0QCxhSToLExJ1ospBHrgtqr78FXwZk0pok8q5XmxY6olMVGO
Bounceable address (for later access): kQCxhSToLExJ1ospBHrgtqr78FXwZk0pok8q5XmxY6olMQxL
(Saved ICO smartcontract creating query to file ico-query.boc)_

Send file ico-query.boc via lite-client to upload smart contract to TON network

#### 2. Enroll bounty stake
    ~/full-node/crypto/fift -I ~/full-node-src/crypto/fift/lib -L Asm.fif -s <this-directory>/fift/send_bounty.fif kQBtkGvKJl918FQv2yezIEMgGDHZXE7P1hN0UhrhRhZmTuLT 12345

Result:

_Create message body to send 12345 bounty stakes to address kQBtkGvKJl918FQv2yezIEMgGDHZXE7P1hN0UhrhRhZmTuLT
B5EE9C7241010201002C00010AA100003039010043800DB20D7944CBEEBE0A85FB64F664086403063B2B89D9FAC26E8A435C28C2CCC9D07C7DDABF
(Saved to file send-bounty-msg.boc)
Use this filename as body message parameter with script wallet.fif_

    ~/full-node/crypto/fift -I ~/full-node-src/crypto/fift/lib -L Asm.fif -s <this-directory>/fift/wallet.fif <this-directory>/fift/owner_wallet kQCxhSToLExJ1ospBHrgtqr78FXwZk0pok8q5XmxY6olMQxL 30 1 -B <this-directory>/fift/send-bounty-msg.boc <this-directory>/fift/send-bounty-query

Result:

_Source wallet address = 0:4fde1eb0ae74609cc545593f69ff07c23ec34bb4424743c7302ee0b13d3d946c 
kQBP3h6wrnRgnMVFWT9p_wfCPsNLtEJHQ8cwLuCxPT2UbDq-
Loading private key from file /fift/owner_wallet.pk
Transferring GR$1. to account kQCxhSToLExJ1ospBHrgtqr78FXwZk0pok8q5XmxY6olMQxL = 0:b18524e82c4c49d68b29047ae0b6aafbf055f0664d29a24f2ae579b163aa2531 seqno=0x1e bounce=-1 
Body of transfer message is x{A100003039}
 x{800DB20D7944CBEEBE0A85FB64F664086403063B2B89D9FAC26E8A435C28C2CCC9D_}
(Saved to file /fift/send-bounty-query.boc)_

Send file send-bounty-query.boc via lite-client to enroll bounty

#### 3. Buy tokens with parameter any_stage = 0:
    ~/full-node/crypto/fift -I ~/full-node-src/crypto/fift/lib -L Asm.fif -s <this-directory>/fift/wallet.fif <this-directory>/fift/investor_wallet kQCxhSToLExJ1ospBHrgtqr78FXwZk0pok8q5XmxY6olMQxL 5 1 -B <this-directory>/fift/current_stage.boc <this-directory>/fift/current-stage-query

Result:

_Transferring GR$1. to account kQCxhSToLExJ1ospBHrgtqr78FXwZk0pok8q5XmxY6olMQxL = 0:b18524e82c4c49d68b29047ae0b6aafbf055f0664d29a24f2ae579b163aa2531 seqno=0x1f bounce=-1 
Body of transfer message is x{00}
signing message: x{0000001F03}
...
(Saved to file  /fift/current-stage-query.boc)_

Send file current-stage-query.boc via lite-client to buy tokens by current stage price only.


#### 4. Finalize ICO:
    ~/full-node/crypto/fift -I ~/full-node-src/crypto/fift/lib -L Asm.fif -s <this-directory>/fift/wallet.fif <this-directory>/fift/owner_wallet kQCxhSToLExJ1ospBHrgtqr78FXwZk0pok8q5XmxY6olMQxL 32 1 -B <this-directory>/fift/finalize_ico.boc <this-directory>/fift/finalize-ico-query

Result:

_Source wallet address = 0:4fde1eb0ae74609cc545593f69ff07c23ec34bb4424743c7302ee0b13d3d946c 
kQBP3h6wrnRgnMVFWT9p_wfCPsNLtEJHQ8cwLuCxPT2UbDq-
Loading private key from file /fift/owner_wallet.pk
Transferring GR$1. to account kQCxhSToLExJ1ospBHrgtqr78FXwZk0pok8q5XmxY6olMQxL = 0:b18524e82c4c49d68b29047ae0b6aafbf055f0664d29a24f2ae579b163aa2531 seqno=0x20 bounce=-1 
Body of transfer message is x{A3}
...
(Saved to file /fift/finalize-ico-query.boc)_

Send file finalize-ico-query.boc via lite-client to finalize ICO.


