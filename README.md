# TON blockchain multisig contract 

Interaction scripts for [original multisig contract](https://github.com/newton-blockchain/ton/blob/master/crypto/smartcont/multisig-code.fc).

## Modifications made to FunC code
Please note that I have [fixed](https://github.com/akifoq/multisig/commit/b5ecf321ffc175c0a29662f2b0134a14d781dd19) a minor bug in a get-method (doesn't affect onchain behaviour) and also [added](https://github.com/akifoq/multisig/commit/cf43cebc88254cb02c8480b0dff9eca431febc4d) additional check for proposed query to prevent infinite repeating of correctly signed by majority of holders, but otherwise incorrect (due to a bug in offchain interface, for instance) order, which could lead to loosing unlimited amount of funds to blockchain fees. Except for this, the code is the same.

## Brief description of multisig wallet
`(n, k)`-multisig wallet is a multisignature wallet with `n` private keys holders, which accepts requests to send messages if the request (aka order, query) collects at least `k` signatures of the holders. An order can have up to three unique messages to send in a single transaction. In contrast to some other blockchains, a holder doesn't need to have own wallet. It is possible to control the multisig using only the keys, while the fee is paid from multisig's own funds. Also multisig supports onchain voting for orders. Namely, if an order sent to multisig has at least one correct signature but less than `k`, then it is saved to contract data to allow collecting missing signatures later. Also any order may have a time limit, after which it is expired. 

### Risks using multisig wallet
Please note that it is not recommended to have more than ~100 non-expired orders simultaneously, because it can lead to out of gas credit exception. An order is considered non-expired from the moment it transefered to the contract to the expiration time (such orders' ids are saved in the contract data to enforce reply-protection).

## Compilation
Compile the contract with `func -SPA stdlib.fc multisig-code.fc -o multisig-code.fif`.

## Scripts
Suppose you have set FIFTPATH variabale to fift library directory. Then you can run the scripts with command `fift -s <script-name> <args>`. When no args are specified, a script displays short help message with info of its usage.

1. You can generate one or several keys with `new-key.fif` script manually, or generate a batch or keys with `generate-keypairs.fif`.
2. Having a text file with list of serialized public keys (in "user-friendly" format) you can create an init message for a new mulstisig wallet with `new-mulstisig.fif` script.
3. After the contract is activated, you may want to create one or several internal messages to send with `create-msg.fif` script.
4. Having message(s) to send you can group it to a new order with `create-order.fif` script.
5. You can add a signature to the order with `add-signature.fif` script, provided you have corresponding private key. Note that the index of the key is the number of line in public keys text file at which it is presented (numbering from 0).
6. Having two orders with the same messages to send, but (potentially) different signatures lists, you can unite the lists with `union-signatures.fif` script.
7. When you want to send the order to the contract, you should create the external message with `create-external-message.fif` script. Note that it requires a private key to ensure authenticity ot the signatures list. Your signature will also be counted as a vote for the order.
8. Send the message and enjoy your order being processed.

Also you can clear signatures list with `clear-signatures.fif` and show indexes of parties signed the order with `show-signed-by.fif`. `show-msg.fif` and `show-order.fif` can be used for displaying messages and orders in human-readable format.
