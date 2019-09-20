## 1: Create Node
```
Note: Depend on currency service, we are can create own Node or use public node.
```
## 2: Create currency Data
```
Note: 
- Some Currencies have same Crypto service.
- Remember config `Withdraw manual threshold`, System will alert to you when withgraw amount is larger than this value
```
## 3: Check Deposit/Withdraw
```
Note: 
- Check Deposit/Withdraw both inside and outside exchange. 
- Check transaction fee, set suitable fee to speed up confirmations on blockchain
```
## 4: Create Master wallet
```
Note: For Currencies have same Crypto service, Create only one Master Wallet model
```
## 5: Create currency pair
```
Note: Remember set some configs as: Min order total, Max order total, Max increase rate, Max decrease rate. These configs are constraints when trading
```
## 6: Init Firebase for currency pair
