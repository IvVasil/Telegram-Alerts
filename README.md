# Telegram-Alerts-Haqq

- First you need to register a bot and get its id and token.
- There is a special bot for this >>>  https://telegram.me/botfather
- Write to him  "/start"
- Then the command to create your bot "/newbot"
- Then come up with a name for your bot and write it. If it is already in use, the bot will ask you to come up with another name.
- If successful, you will see information with your token data for accessing the HTTP API
- Write down the token data, they will be useful to us later
- Visit your bot and click start
- Now we need to find your telegram account ID
- Visit the bot and click start. https://telegram.me/getmyid_bot
- Now we just need to configure the alert script.


##### 1. Create a folder and go to it
```
mkdir $HOME/Telegram-Alerts-Haqq; cd $HOME/Telegram-Alerts-Haqq
```


#### 2. Install the nano editor and create a file with the following content
```
sudo apt install nano
nano $HOME/Telegram-Alerts-Haqq/Telegram-Alerts-Haqq.sh
```
```
#!/bin/bash

# Node name, e.g. "Cosmos"
NODE_NAME=""
# File name for saving parameters, e.g. "cosmos.log"
LOG_FILE="$HOME/Telegram-Alerts-Haqq/Telegram-Alerts-Haqq.log"
# Your node RPC address, e.g. "http://127.0.0.1:26657"
NODE_RPC="http://127.0.0.1:26657"
# Trusted node RPC address, e.g. "https://rpc.cosmos.network:26657"
SIDE_RPC="https://haqq-t.rpc.manticore.team"
# Telegram bot API
TG_BOT=
# Telegram chat ID
TG_ID=
ip=$(wget -qO- eth0.me)

touch $LOG_FILE
REAL_BLOCK=$(curl -s "$SIDE_RPC/status" | jq '.result.sync_info.latest_block_height' | xargs )
STATUS=$(curl -s "$NODE_RPC/status")
CATCHING_UP=$(echo $STATUS | jq '.result.sync_info.catching_up')
LATEST_BLOCK=$(echo $STATUS | jq '.result.sync_info.latest_block_height' | xargs )
VOTING_POWER=$(echo $STATUS | jq '.result.validator_info.voting_power' | xargs )
ADDRESS=$(echo $STATUS | jq '.result.validator_info.address' | xargs )
source $LOG_FILE

echo 'LAST_BLOCK="'"$LATEST_BLOCK"'"' > $LOG_FILE
echo 'LAST_POWER="'"$VOTING_POWER"'"' >> $LOG_FILE

curl -s "$NODE_RPC/status"> /dev/null
if [[ $? -ne 0 ]]; then
    MSG="$ip node is stopped!!!"
    MSG="$NODE_NAME $MSG"
    SEND=$(curl -s -X POST -H "Content-Type:multipart/form-data" "https://api.telegram.org/bot$TG_BOT/sendMessage?chat_id=$TG_ID&text=$MSG"); exit 1
fi


if [[ $LAST_POWER -ne $VOTING_POWER ]]; then
    DIFF=$(($VOTING_POWER - $LAST_POWER))
    if [[ $DIFF -gt 0 ]]; then
        DIFF="%2B$DIFF"
    fi
    MSG="$ip voting power changed  $DIFF%0A($LAST_POWER -> $VOTING_POWER)"
fi

if [[ $LAST_BLOCK -ge $LATEST_BLOCK ]]; then

    MSG="$ip node is probably stuck at block $LATEST_BLOCK"
fi

if [[ $VOTING_POWER -lt 1 ]]; then
    MSG="$ip validator inactive\jailed. Voting power $VOTING_POWER"
fi

if [[ $CATCHING_UP = "true" ]]; then
    MSG="$ip node is unsync, catching up. $LATEST_BLOCK -> $REAL_BLOCK"
fi

if [[ $REAL_BLOCK -eq 0 ]]; then
    MSG="$ip can't connect to  $SIDE_RPC"
fi

if [[ $MSG != "" ]]; then
    MSG="$NODE_NAME $MSG"
    SEND=$(curl -s -X POST -H "Content-Type:multipart/form-data" "https://api.telegram.org/bot$TG_BOT/sendMessage?chat_id=$TG_ID&text=$MSG")
fi

```
### Substitute your data in the script in !!!!! TG_BOT=(Telegram bot API) and TG_ID=(Telegram ID)
Also specify the name of your validator NODE_NAME=""

```
sudo chmod 744 Telegram-Alerts-Haqq.sh; cd
```

#### Let's just put the script in Cron
```
 crontab -e
 ```
 
 #### We will make the script run every minute.Add this line at the end
  ```
 */5 * * * *  /bin/bash $HOME/Telegram-Alerts-Haqq/Telegram-Alerts-Haqq.sh
  ```
 
 

