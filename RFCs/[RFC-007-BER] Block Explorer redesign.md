# Block Explorer redesign 


## Stories

|##|Story|
|---|----------|
|Story#1 | As a SN owner I want to see in the Tx List in the Blockchain Explorer info (or marker) about pool and type of each  Tx: RTA, Stake, Fee, Reward|
|Story#2 |As a SN owner I want to see SN List in the Blockchain Explorer  and want to have the ability to search my SN in the SN List
|Story#3 |As a SN owner I want to see BBList in the Blockchain Explorer  and want to have the ability to search my SN in the BBList
|Story#4 |As a SN owner I want to see AuthSampleList in the Blockchain Explorer  and want to have the ability to search my SN in the AuthSampleList
|Story#5 |As a SN owner I want to see DisqList in the Blockchain Explorer  and want to have the ability to search my SN in the DisqList
|Story#6 |As a Graft network user  I want to have the ability to see details of RTA  transactions



## Use cases

### Story#1: As a SN owner I want to see in the Tx List in the Blockchain Explorer info (or mark) about type of Tx: RTA, Stake, Fee, Reward

1.1. Go to Blockchain Explorer: https://blockexplorer.graft.network/

1.2. Additional column “pool” (see pic.1) should be present in the block “Transactions in the last 11 blocks” 

   1.2.1. This column should include info about pool for a specific Tx (see pic.1[1]).
     
   1.2.2 The pool information should contain a link by clicking on which I can go to the site of the corresponding pool.

   1.2.3. For example: 
       
   - Graft.community - https://pool.graft.community/#pool_blocks
       
   - Cryptopool.space -  https://grft.cryptopool.space/#/blocks
       
   - Hash Vault - https://graft.hashvault.pro/en/#!/blocks

   1.2.4 Info should be displayed in the same format with other info in these tables (font, size etc)

1.3 Column “transaction hash” in the blocks “Transaction pool” and “Transactions in the last 11 blocks” should contain marker for specific Tx (see pic.1[2]).

![BE_22](https://user-images.githubusercontent.com/45132833/61708384-5ddb7680-ad55-11e9-9334-b65584ae644c.png)

Pic.1


   
   1.3.1 Following transactions are marked:
   
   - RTA Tx
   - Stake Tx ( and additional marker shows if it Tx is in stake) 
   - Disq Tx
   - Stimulus Tx
   
   > NOTE: Normal tx is not marked

   1.3.2  When the cursor hover on the Tx marker, a pop-up window should appear with additional info.
   
   1.3.3  Examples of markers and additional Info in a pop-up window:
   
   Table 1

|Tx type|Markers example (can be change to other)|Additional Info in a pop-up window|
|---|----------|------|
|Normal Tx| - | - |
|RTA Tx|![rta_1](https://user-images.githubusercontent.com/45132833/61711490-ddb90f00-ad5c-11e9-8155-55fe7521544e.png)|RTA transaction|
|Stake Tx|![stake_0](https://user-images.githubusercontent.com/45132833/61710438-5c607d00-ad5a-11e9-8be9-e2217bf033b0.gif)|SN Stake transaction partly in stake|
|Stake Tx (if  Tx amount>=stake amount)|![stake_1](https://user-images.githubusercontent.com/45132833/61710967-97af7b80-ad5b-11e9-927c-c25660c4d079.png)  ![stake_0](https://user-images.githubusercontent.com/45132833/61710438-5c607d00-ad5a-11e9-8be9-e2217bf033b0.gif)|SN Stake transaction (T1)|
| Disq Tx|![DIsq](https://user-images.githubusercontent.com/45132833/61711291-5e2b4000-ad5c-11e9-9391-ab50b19e4da2.png) |Disqualification transaction| 
| Fee Tx|![Fee](https://user-images.githubusercontent.com/45132833/61710751-004a2880-ad5b-11e9-8ddd-01cf6d9b3716.png) |Stimulus Transaction|


### Story#2: As a SN owner I want to see SN List in the Blockchain Explorer  and want to have the ability to search my SN in the SN List

2.1. Go to Blockchain Explorer: https://blockexplorer.graft.network/

2.2 Block “SN List” should be present after block “Transactions in the last 11 blocks”  with default info about SN List in last block (pic.2,[3])

2.3. Following info from SN List  should displayed :
   
   - Height
   - Public ID
   - IsStakeValid (True/Faulse)
   - Tier (0/1/2/3/4)
   - StakeAmount
   - StakeFirstValidBlock
   - StakeExpiringBlock

2.4. By default I can see info about 11 SNs. For displaying more info I should press button “> more” on the end of SN List.

2.5. By default I can see SN List for the last block.  

2.6. When I enter information into the field (pic.2[1]) and press the Search button I should see info about  SNs in block  which I search

2.7. When I enter information into the field (pic.2[2]) and press the Search button I should see info about  SN which I search 

2.8. When I enter information into the fields (pic.2[1]) and (pic.2[2]) and press the Search button I should see info about  my SN for the block which I search 

2.9. The public_ID in the SN List should contain a link by clicking on which I can go to the new screen with SN's details (pic.3)

  2.9.1. By default, the screen displays information about the Supernode selected in the previous mode

  2.9.2. Additional search is provided by SN public ID and block height. If no block height is specified, information is searched for the last SN Stake

  2.9.3. The Transaction hash  in the Transaction list should contain a link by clicking on which I can go to the new screen with Tx's details (pic.4)

![2019-07-10_16-04-13](https://user-images.githubusercontent.com/45132833/61707923-346e1b00-ad54-11e9-9af2-ca28476528fe.jpg)

Pic.2


### Story#3: As a SN owner I want to see BBList in the Blockchain Explorer  and want to have the ability to search my SN in the BBList

3.1. Go to Blockchain Explorer: https://blockexplorer.graft.network/

3.2 Block “BB List” should be present after block “Supernodes List”  with default info about BB List in last block (pic.2,[4])

3.3. Following info from BB List  should displayed :

   - Height
   - Public ID
   - Tier (0/1/2/3/4)
   - StakeAmount

3.4. By default I can see info about 11 SNs. For displaying more info I should press button “> more” on the end of BB List.

3.5. By default I can see BB List for the last block. 

3.6. When I enter information into the field (pic.2[1]) and press the Search button I should see info about  SNs in the BBList in the block  which I search 

3.7. When I enter information into the field (pic.2[2]) and press the Search button I should see info about  SN  in the BBList which I search 

3.8. When I enter information into the fields (pic.2[1]) and (pic.2[2]) and press the Search button I should see info about  my SN  in the BBList for the block which I search.

3.9. The public_ID in the BB List should contain a link by clicking on which I can go to the new screen with SN's details (pic.3).

3.9.1. By default, the screen displays information about the Supernode selected in the previous mode

3.9.2. Additional search is provided by SN public ID and block height. If no block height is specified, information is searched for the last SN Stake

3.9.3. The Transaction hash  in the Transaction list should contain a link by clicking on which I can go to the new screen with Tx's details (pic.4)

![2019-07-11_01](https://user-images.githubusercontent.com/45132833/61707924-3506b180-ad54-11e9-806c-396161395b96.jpg)

Pic.3

### Story#4: As a SN owner I want to see AuthSampleList in the Blockchain Explorer  and want to have the ability to search my SN in the AuthSampleList

4.1. Go to Blockchain Explorer: https://blockexplorer.graft.network/

4.2 Block “AuthSample List” should be present after block “BB List”  with default info about AuthSample List in last block (pic.2,[5])

4.3. Following info from AuthSample List  should displayed :

   - Height
   - Payment ID (for RTA TX)
   - Public ID
   - Tier (0/1/2/3/4)
   - StakeAmount
   - StakeFirstValidBlock
   - StakeExpiringBlock

4.4. By default I can see AuthSample List for the last block. 

4.5. When I enter information into the field (pic.2[1]) and press the Search button I should see info about  SNs in the AuthSample List in the block  which I search 

4.6. When I enter information into the field (pic.2[2]) and press the Search button I should see info about  SN  in the AuthSample List which I search 

4.7. When I enter information into the fields (pic.2[1]) and (pic.2[2]) and press the Search button I should see info about  my SN  in the AuthSample List for the block which I search 

4.8. The public_ID in the AuthSample List should contain a link by clicking on which I can go to the new screen with SN's details (pic.3).

   4.8.1. By default, the screen displays information about the Supernode selected in the previous mode.
 
   4.8.2. Additional search is provided by SN public ID and block height. If no block height is specified, information is searched for the last SN Stake.

   4.8.3. The Transaction hash  in the Transaction list should contain a link by clicking on which I can go to the new screen with Tx's details (pic.4).

![2019-07-11_02](https://user-images.githubusercontent.com/45132833/61707925-3506b180-ad54-11e9-933a-0baa98691108.jpg)

Pic.4


### Story#5: As a SN owner I want to see DisqList in the Blockchain Explorer  and want to have the ability to search my SN in the DisqList

5.1. Go to Blockchain Explorer: https://blockexplorer.graft.network/

5.2 Block “Disq List” should be present after block “AuthSample List”  with default info about Disq List in last block (pic.2,[6])

5.3. Following info from Disq List  should displayed :

   - Public ID
   - StakeAmount
   - Disq Tx hash
   - RTA Tx hash (for  disq Tx #2)
   - StakeFirstValidBlock
   - StakeExpiringBlock
   - Disq ExpiringBlock
   - Disq StartBlock

5.4. By default I can see info about 11 SNs. For displaying more info I should press button “> more” on the end of Disq List.

5.5. By default I can see Disq List for the last block. 

5.6. When I enter information into the field (pic.2[1]) and press the Search button I should see info about  SNs in the DisqList in the block  which I search 

5.7. When I enter information into the field (pic.2[2]) and press the Search button I should see info about  SN  in the DisqList which I search 

5.8. When I enter information into the fields (pic.2[1]) and (pic.2[2]) and press the Search button I should see info about  my SN  in the DisqList for the block which I search 

5.9. The public_ID in the Disq List should contain a link by clicking on which I can go to the new screen with SN's details (pic.3).

   5.9.1. By default, the screen displays information about the Supernode selected in the previous mode

   5.9.2. Additional search is provided by SN public ID and block height. If no block height is specified, information is searched for the last SN Stake

   5.9.3. The Transaction hash  in the Transaction list should contain a link by clicking on which I can go to the new screen with Tx's details (pic.4)


### Story#6: As a Graft network user  I want to have the ability to see details of RTA  transactions

6.1. Pressing  on the “Transaction hash” link I should go to the transaction detail Screen (pic.4)

6.2. Additional info about RTA transaction is displayed in the “RTA info” tab (pic.4).

6.3. Following RTA info is displayed:

   - Height for AuthSampleList
   - RTA payment_id
   - Info about SNs from AuthSampleList:
     - SN Public ID
     - Is RTA Tx Signed by this SN
     - Tier
     - Stake Amount
     - StakeFirstValidBlock
     - StakeExpiringBlock.
