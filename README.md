oclvanityminer-cheater
-----

Proof of concept "block withholding attack" on a vanity address generation pay-per-share pool.  
No known vulnerable pools exist nor are any known to have ever existed!  Lets keep it that way...  
NOTE: This repo is simply a fork of my vanitygen-plus repo with a modified oclvanityminer.c file.  
This repo is NOT meant for actual use!  Only as a proof of concept.  

DISCLAIMER: I will not be held liable for your actions, DO NOT CHEAT A POOL, this code is being uploaded only as a proof of concept, NOT FOR CHEATING A POOL, if someone creates a pool that meets all the conditions to be vulnerable I can not be held liable for your actions.  DO NOT TRY TO USE THIS TO CHEAT A POOL, YOU WILL BE WASTING YOUR TIME AND RESOURCES!  I do not condone illegal activity or cheating!  And if I find that a vulnerable pool exists I will notify them immediately to correct the problem and delete the repo! This code is mainly here for devs of vanity pools to test if their pool is vulnerable although they should immediately know after reading the readme and to keep vanity mining secure for everyone.  Normally for such a big attack I would upload the code as broken but since vanity mining is such a rare and niche market and no vulnerable pools currently exist I have left the application fully in tact and functional as it is unlikely that such an attack will ever be possible.  Although one measure has been taken to prevent abuse, to run the theoretical attack you do have to supply the application with a "magic word".  If you are a pool developer I would be more than willing to supply you with this information.  

The attack works in one of three ways.  

1) You mine on a Pay-Per-Share pool, you don't find the vanity address but the pool does, you get paid your shares.  
2) You mine on a PPS pool, you find the vanity address, but you don't submit it to the PPS pool, you withhold the "block", then you submit it to the cheater pool, you now claim the entire reward instead of only the shares you contributed.  
3) You mine on a PPS pool, someone outside the pool finds the vanity address, you don't get paid.  

Several odd conditions need to exist for this vulnerability to work.  

1) There needs to be a parent/mother vanity pool setup for solo mining. (used as our cheater pool)  
2) There needs to be a merge mining PPS pool that gets addresses to mine from the parent pool's /getWork.  
3) The PPS pool then has to accept solutions using the same PUBKEY as the parent pool's /getWork.  In other words the PPS pool can't add any additional layers of encryption such as a second split-key layer where they would generate a new PUBKEY for the getWork.  This is where the attack gets thwarted in real life, any pool that meets the above two criteria do not meet this third condition.  If you are running a pool that meets the above two criteria be sure to encrypt everything on your end in turn generating additional pub/partprivkeys known only to the pool so this way your getWork can not be submitted to the parent pool unless first sent through your pool to be decrypted then in turn sent to the parent pool.  For more information on how a split key of a split key works by adding the two public keys scroll to the end of the readme.  

To run the attack:  
`./oclvanityminer -c 1PREFIXTOCHEAT -u https://URL-OF-PPS-POOL -a 1BTC-PAYMENT-ADDRESS-FOR-PPS-POOL -i 1 -x https://URL-OF-CHEATER-POOL -y 1BTC-PAYMENT-ADDRESS-FOR-CHEATER-POOL`  

To compile from source:  
`gpg --decrypt oclvanityminer.c.gpg>oclvanityminer.c`  
`make oclvanityminer`  

```
oclVanityMiner PLUS CHEATER (OpenSSL 1.0.2k  26 Jan 2017)  
Usage: ./oclvanityminer -u <URL> -a <credit address>  
Organized vanity address mining client using OpenCL.  Contacts the specified  
bounty pool server, downloads a list of active bounties, and attempts to  
generate the address with the best difficulty to reward ratio.  Maintains  
contact with the bounty pool server and periodically refreshes the bounty  
list.  
By default, if no device is specified, and the system has exactly one OpenCL  
device, it will be selected automatically, otherwise if the system has  
multiple OpenCL devices and no device is specified, an error will be  
reported.  To use multiple devices simultaneously, specify the -D option for  
each device.  

Options:  
-u <URL>      Bounty pool URL  
-a <address>  Credit address for completed work  
-i <interval> Set server polling interval in seconds (default 90)  
              TIP: For pay-per-share vanity pools, set interval to  
                   1 to process maximum shares!  
-c <prefix>   Set a cheater prefix,  
              meaning do not submit this prefix to the main pool!  
              Instead results for this prefix will be submitted to  
              a secondary pool that can be set using the -x flag.  
              NOTE: The cheater option only works if both pools  
              share the same public key.  In this instance one can  
              merge mine on a vanity pool collecting shares.  
              If the pool finds the address you get paid out your  
              shares, but if you find the address you submit it to  
              a secondary solo pool and claim the entire reward.  
              THIS IS CHEATING! This option is included here only  
              as a proof of concept!  Do not use, although there  
              is no known pool that is currently vulnerable to such an attack.  
              If such a pool exists we take no responsibility for you using  
              this program in any manner illicit or otherwise. 
              HINT: Don't forget to say the magic word!   
-x <url>      URL to submit the cheater prefix to.  
              This would likely be a solo vanity mining pool with a higher  
              payout than the merge mining pool.  
-y <address>  BTC address to submit with the cheater prefix URL for payment.  
              This is the address to be paid by the secondary pool when  
              submittinga cheater prefix.  
-v            Verbose output  
-q            Quiet output  
-p <platform> Select OpenCL platform  
-d <device>   Select OpenCL device  
-D <devstr>   Use OpenCL device, identified by device string  
              Form: <platform>:<devicenumber>[,<options>]  
              Example: 0:0,grid=1024x1024  
-S            Safe mode, disable OpenCL loop unrolling optimizations  
-w <worksize> Set work items per thread in a work unit  
-t <threads>  Set target thread count per multiprocessor  
-g <x>x<y>    Set grid size  
-b <invsize>  Set modular inverse ops per thread  
-V            Enable kernel/OpenCL/hardware verification (SLOW)  
-m <minvalue> Set minimum value (in BTC/Gkey) for the miner to accept work  
```
  
TEST FOR SECURING THE CHILD PPS POOL USING A SPLIT KEY OF A SPLIT KEY  
-----  
  
PREFACE: Obviously this is just a test and the parent pool should never have the end users private part key, the parent pool only gets the end users public key.  This is all set up for testing, showing how to use a split key of a split key by adding two public keys.  
  
Steps  
1) Generate two wallets for the parent and child pool using `./keyconv -G`  
2) Add the public keys  
3) Mine the vanity address using getWork consisting of the combined public key  
4) Add the mined priv key part to child pool's priv key part to generate a new privkey  
5) Add resulting privkey to the parent pool's priv key part to generate the end users privkey  
  
GENERATE SPLIT KEY WALLETS WITH KEYCONV  
  
PARENT POOL WALLET:  
(The user requesting the vanity address is the one that does this for the parent pool and they submit only their pubkey) 
`./keyconv -G`  
>Pubkey (hex): 04b62c23bf6cce4641877098ccff8452d2db2689451b52f15b5d47faf89faf5c605fcc74cd0123f4b6e77f6960f564c260bf65089a376202bd54d148a962ed5766  
>Privkey (hex): B1DDC32FE84CD5E962276734A96D59954D08CACF85F9816A1A67BB5A672A8F  
>Address: 1HdxXbBPQKykuTufH37zJUZg7S647aRD7h  
>Privkey: 5HpbL2yQTfQ9R4q5ssXKFfUWXdKeCajYSi6woS6JWMaYY8uXECv  
  
CHILD PPS POOL WALLET:  
`./keyconv -G`  
>Pubkey (hex): 0437c4f8e2f9ddf046b4753587bc688678dfe0c0c99ffd37f1e21ea6287a5482a0a0e9f5860d618551fa9e04018ab469fe4a7efa2c901b11fff32805fc13ce6349  
>Privkey (hex): 38F55D88EF19E390C9194C679C676D47B75D7EA633B530889B25C71CDB8B760D  
>Address: 1H96qe5EoVTkrVmn2uuyCJ1BSm6rRyS87m  
>Privkey: 5JFNWU9BQMwAZwHPyh53vBM3RPTbz5jZdQf5kqRNU48MNfRJBrT  
  
ADD BOTH POOLS PUB KEYS TO GET:  
>0477A21EA8C8ED5D5A49BD2AB198695D52F19FBBF80930001948929CB21402A294C58C4E7FA0E04CE10DA6A1809102B1F07A2F8E50DFD4B586046AD5D0557090ED  
Use "vanity wallet" tool from https://www.bitaddress.org  (You can do all these tests on their site if you wish)  
ADD THIS TO /getWork  
  
USE SELF HOSTED getWork FOR TESTING: 
>127.0.0.1/getWork  
>1NRcvA:0477A21EA8C8ED5D5A49BD2AB198695D52F19FBBF80930001948929CB21402A294C58C4E7FA0E04CE10DA6A1809102B1F07A2F8E50DFD4B586046AD5D0557090ED:0:0.000000;  
  
RUN OCLVANITYMINER  
`./oclvanityminer -u 127.0.0.1 -a ADDR`  
  
MINED PRIV PART KEY FOUND FOR PREFIX 1NRcvA:  
>5HtYxxhqurRHrdWFVVTLir5LgsVcNXnmG65EAW1W8JquZ71Yqm1  
  
ADD MINED PRIV PART KEY FOUND ABOVE WITH CHILD POOLS PRIV PART KEY:  
`./keyconv -c 5HtYxxhqurRHrdWFVVTLir5LgsVcNXnmG65EAW1W8JquZ71Yqm1 5JFNWU9BQMwAZwHPyh53vBM3RPTbz5jZdQf5kqRNU48MNfRJBrT`  
>Address: 16a5n7fq8p1B46XGo3fHKTMuuCr2Npm48p  
>Privkey: 5JKdtkPwEknUAZQNRz5VYygFtf8J9UfFAgfXhAP8MV7cm1SFiM8  

  
ADD RESULT WITH PARENT POOL(END USERS) PRIV PART KEY:  
`./keyconv -c 5JKdtkPwEknUAZQNRz5VYygFtf8J9UfFAgfXhAP8MV7cm1SFiM8 5HpbL2yQTfQ9R4q5ssXKFfUWXdKeCajYSi6woS6JWMaYY8uXECv`  
>Address: 1NRcvAiRtmdaRK396VCdL2gmvPHkmJdn7G  
>Privkey: 5JKwe6vFcxcdKcrBGf9uibQeCgd28vXhtahhGRRgcxqX8aCWMo4  

CHECK ADDRESS WITH KEYCONV USING END USERS PRIV KEY:
`./keyconv 5JKwe6vFcxcdKcrBGf9uibQeCgd28vXhtahhGRRgcxqX8aCWMo4`  
>Address: 1NRcvAiRtmdaRK396VCdL2gmvPHkmJdn7G  
>Privkey: 5JKwe6vFcxcdKcrBGf9uibQeCgd28vXhtahhGRRgcxqX8aCWMo4  