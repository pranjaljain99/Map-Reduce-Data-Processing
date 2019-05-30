# Map-Reduce-Data-Processing
BITCOIN ANALYSIS USING BIG DATA JOBS :- ANALYSIS OF BITCOIN TRANSACTIONS

# Goal : 
Apply the techniques covered in Big Data Processing to analyse a subset of the bitcoin blockchain, ranging from the first mined bitcoin in 2009 to December 2014. . 

There are many resources available for understanding bitcoin and the block chain, a good place to start is the original Bitcoin paper. There are also may sites dedicated to providing information about individual blocks, transactions and wallets, such as blockexplorer and blockchain.com.

# DATASET OVERVIEW
Bitcoin is a decentralised digital currency that enables instant payments to anyone, anywhere in the world. Bitcoin uses public-key cryptography, peer-to-peer networking, and proof-of-work to process and verify payments. Bitcoins are sent from one address (wallet) to another, with each transaction broadcast to the network and included in the blockchain so that the bitcoins cannot be spent twice.

A subset of the blockchain (Blocks 1 to ~330000) has been collected and saved to HDFS at /data/bitcoin. These blocks have been changed from their original raw json format by splitting them into five comma delimited csv files: blocks.csv; transactions.csv; coingen.csv (Coin Generations/Mined coins); vin.csv; and vout.csv. To explain what these files contain a description and schema can be found below:

# DATASET DESCRIPTION
At any one point in time one bitcoin block exists, linked to a mathematical problem. While miners race to solve this problem the block records all transactions taking place while it remains unsolved (transactions.csv). Each time a block is completed it becomes a permanent record of these past transactions and gives way to a new block in the blockchain (blocks.csv).

When a winning miner solves a block's problem, the answer is shared with other mining nodes and validated. Every time a miner solves a problem a set amount of BTC (Bitcoin currency symbol) is awarded to the miner and enters circulation (coingen.csv). The first record in the next block is a transaction that awards the winning miner this newly minted BTC.

Each transaction tracks the bitcoin amounts being transferred, the source transaction(s) of the coins (vin.csv) and the destination wallet(s) (vout.csv). This allows each BTC to be tracked from the latest transaction it was involved in, right back to when it was minted.

# DATASET SCHEMA
BLOCKS.CSV
height: The block number
hash: The unique ID for the block
time: The time at which the block was created (Unix Timestamp in seconds)
difficulty: The complexity of the math problem associated with the block

# Sample entry:

    0, 000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f,  1231006505,  1
TRANSACTIONS
tx_hash: The unique id for the transaction

blockhash: The block this transaction belongs to

time: The time when the transaction occurred

tx_in_count: The number of transactions with outputs coming into this transaction

tx_out_count: The number of wallets receiving bitcoins from this transaction

Sample entry:

    0e3e2357e806b6cdb1f70b54c3a3a17b6714ee1f0e68bebb44a74b1efd512098,00000000839a8e6886ab5951d76f411475428afc90947ee320161bbf18eb6048,1231469665,1,1
COINGEN
db_id: The block the coin generation was found within (maps to block height not hash)

tx_hash: Transaction id for the Coin Generation

coinbase: Input of a generation transaction. As a generation transaction has no parent and creates new coins from nothing, the coinbase can contain any arbitrary data (or perhaps a hidden meaning).

sequence: Allows unconfirmed time-locked transactions to be updated before being finalised

Sample entry:

    1,  0e3e2357e806b6cdb1f70b54c3a3a17b6714ee1f0e68bebb44a74b1efd512098,  04ffff001d0104,  4294967295
VIN
txid: The associated transaction the coins are going into

tx_hash: The transaction the coins are coming from

vout: The ID of the output from the previous transaction - the value equals n in vout below

Sample entry:

    f4184fc596403b9d638783cf57adfe4c75c605f6356fbc91338530e9831e9e16,0437cd7f8525ceed2324359c2d0ba26006d92d856a9c20fa0241106ee5a597c9,0
VOUT
hash: The associated transaction

value: The amount of bitcoins sent

n: The id for this output within the transaction. This will equal the vout field within the vin table above, if the coins have been respent

publicKey: The id for the wallet where the coins are being sent

Sample entry:

    0e3e2357e806b6cdb1f70b54c3a3a17b6714ee1f0e68bebb44a74b1efd512098, 50, 0, {12c6DSiU4Rq3P4ZxziKxzrL5LmMBrzjrJX}
ASSIGNMENT
Write a set of jobs using either Map/Reduce or Spark that process the given input and generate the data required to answer the following questions: (You can mix and match, for example using MRJob for part A and Spark for part B/C)

# PART A. TIME ANALYSIS
Create a bar plot showing the number of transactions which occurred every month between the start and end of the dataset. What do you notice about the overall trend in the utilisation of bitcoin?

Note: As the dataset spans multiple years and you are aggregating together all transactions in the same month, make sure to include the year in your analysis.

# PART B. TOP TEN DONORS
Obtain the top 10 donors over the whole dataset for the Wikileaks bitcoin address: {1HB5XMLmzFVj8ALj6mfBsbifRoD4miY36v}. Is there any information on who these wallets belong to? Can you work out how much was being donated if converted into pounds?

Note: To extract the top ten you will need to join the v_in and v_out files together to get the wallet ID's and BTC amount coming in and out of relevant transactions. However, these are both large files so joining them straight away would be a very resource intensive job. In order to guide this part of the coursework, we provide in the rest of this section a potential structure.

Initial Filtering: In order to speed up the join operation, first filter /data/bitcoin/vout.csv to only rows containing the wallet of interest, {1HB5XMLmzFVj8ALj6mfBsbifRoD4miY36v} (make sure to include the curly brackets).
Copy the results from HDFS back to your local directory, as it will be needed in the next step. It should look similar to the following, only containing transactions where Wikileaks was a recipient:

    "0008291614a680a9d7c13bed40e82a0a8122dfcdf1faa8c61513bc9dd7ed5c61,0.0154,1,{1HB5XMLmzFVj8ALj6mfBsbifRoD4miY36v}"
    "0081b26044d7282245c574d57a93942df1b59ac3d51194f8bde1869f91acb2b6,2,1,{1HB5XMLmzFVj8ALj6mfBsbifRoD4miY36v}"
    "00911db46added579bf47794b3140a197fa0c1501d77f9105f0e70695deef671,0.1,0,{1HB5XMLmzFVj8ALj6mfBsbifRoD4miY36v}"
    "00a4c33fe09cf8b7c8a188c19af5af4fdaf73bd08a4838145bfe14b445e736f0,0.40096182,0,{1HB5XMLmzFVj8ALj6mfBsbifRoD4miY36v}"
First Join: Once you have obtained this much smaller version of the v_out file, the next step is to perform a replication join between it and /data/bitcoin/vin.csv (as seen in lab 4).
Place all the transaction hashes within the v_out file into a dictionary and compare these to the txid field within each v_in. Yield all rows which are involved in one of these transactions.

Again merge the HDFS output to your local directory. As bitcoin transactions may have many inputs and outputs you will see the number of rows has increased, with possibly several v_in's for each of the v_out's found in the first sub-task.

Second join. As a v_in only specifies the previous transaction where the bitcoins originated from, this second output must be rejoined with /data/bitcoin/vout.csv.
Store the tx_hash and vout values from the v_in file into a dictionary and match these to the hash and n fields (respectively) within the full v_out file. When you have a match on both, yield the publicKey (wallet) and value in a pair to the reducer.

Top ten. Finally, sort these pairs via a top ten reducer.
Note: There is no way to tell which v_in links to which v_out, so assume all input addresses are sending their money to Wikileaks even if the amount of bitcoins sent was higher than the received amount.

# PART C. DATA EXPLORATION
Explore how much of BTC is used as a means of exchange (small transactions) as opposed to long term investment (Parked coins or large purchases)

Identify different types of users e.g."whales" (volume trader/"hodler" of BTC), merchant accepting BTCs, or possibly criminal organizations. 

Ransomware often gets victims to pay via bitcoin. Find wallet IDs involved in such attacks and investigate how much money has been extorted. What happens to the coins afterwards?

Find a dataset containing the prices of bitcoin throughout time (Many sites exist with such information). Utilising the Spark MlLib library, combine this, characteristics extracted from the data (volume of bitcoins sold per day, transactions per pay), and any other external sources into a model for price forecasting. How far into the future from our subset end date (September 2013) is your model accurate? How does the volatility of bitcoin prices affect the effectiveness of your model?
