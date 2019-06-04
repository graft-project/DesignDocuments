# GRAFT 2/2 Multisig Light Wallet
## Components
1. Wallet Backend (WB)
2. Light Client Wallet (LCW)

## Wallet Creation
1. LCW does next operations:

   a. generates multisig data (analogue to CLI prepare_multisig command)

   b. sends multisig data to the WB

2. When WB receives multisig data from LCW, it:
   
   a. generates own multisig data (analogue to CLI prepare_multisig command)
  
   b. sends these multisig data to the LCW
  
   c. creates 2/2 multisig wallet using LCW multisig data (analogue to CLI make_multisig command).
  
3. When LCW receives multisig data from WB, it 

   a. creates 2/2 multisig wallet using these multisig data (analogue to CLI make_multisig command)

   b. exports key images (analogue to CLI export_multisig_info command)

   c. sends key images to WB

4. When WB receives key images data, it

   a. imports key images (analogue to CLI import_multisig_info command)

   b. stores wallet data.

## Balance Updating
1. If LCW wants to update its balance, it sends the request to the WB.
2. When WB receives a balance update request, it refreshes data from blockchain and returns the result to LCW
3. LCW receives a WB response and stores the updated balance.

## Transfer
>**_Note:_ LCW and WB should be always synced!**

1. To spend money, LCW should

   a. send the transfer request with receiver wallet address, transfer amount and payment id or subaddress (payment id and subaddress are optional) to WB

2. When WB receives the transfer request, it

   a. creates a partially signed transaction (analogous to CLI transfer command)

   b. exports key images (analogue to CLI export_multisig_info command)

   c. stores wallet data

   d. send an unsigned transaction and exported key images to LCW

3. When LCW receives an unsigned transaction and key images, it

   a. signs transaction (analogue to CLI sign_multisig command)

   b. imports received key images (analogue to CLI import_multisig_info command)

   c. exports its key images (analogue to CLI export_multisig_info command)

   d. stores wallet data

   e. sends signed transaction and exported key images to WB

4. When WB receives signed transaction and key images from LCW, it

   a. submits the transaction to the blockchain

   b. imports key images (analogue to CLI import_multisig_info command)

   c. stores wallet data

   d. returns successful status to the LCW.
