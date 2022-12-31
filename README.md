# amboss.space healthcheck for CLN Nodes
How to add a healthcheck to your node's amboss page. This tutorial is based on raspiblitz's environment, but should work similar on other platforms. Watch out for users and file paths.

## Create a short script file

On your node create a new file in your bitcoin's folder

```
sudo nano /home/bitcoin/amboss-ping.sh
```

and paste in the following lines

```
#!/bin/bash
# Sends a health check ping to the Amboss API. 
# Docs: https://docs.amboss.space/api/monitoring/health-checks
URL="https://api.amboss.space/graphql"
NOW=$(date -u +%Y-%m-%dT%H:%M:%S%z)
echo "Timestamp: ${NOW}"
SIGNATURE=$(/home/bitcoin/lightning/cli/lightning-cli signmessage "$NOW" | jq -r .zbase)
echo "Signature: ${SIGNATURE}"
JSON="{\"query\": \"mutation HealthCheck(\$signature: String!, \$timestamp: String!) { healthCheck(signature: \$signature, timestamp: \$timestamp) }\", \"variables\": {\"signature\": \"$SIGNATURE\", \"timestamp\": \"$NOW\"}}"
echo "Sending ping..."
echo "$JSON" | curl -sSf --data-binary @- -H "Content-Type: application/json" -X POST $URL
```

save by pressing CTRL + X, Y and enter.

## Make it executable

Log into super user by typing

```
su
```

and give proper rights to the file

```
chmod +ax /home/bitcoin/amboss-ping.sh
```

this will make it executable for all users (inclusive 'bitcoin')

type `exit` to get back to user 'admin'

## Start the script

Type

```
sudo -u bitcoin /home/bitcoin/amboss-ping.sh
```

to run and execute the script. 

## Make it run automatically

To make it run automatically, we have to create a cronjob for the user 'bitcoin'

```
sudo -u bitcoin crontab -e
```

at the bottom of the file paste

```
3,8,13,18,23,28,33,38,43,48,53,58 * * * * /home/bitcoin/amboss-ping.sh
```

and save by CTRL + X, Y and enter. This will run the script at 00:03, 00:08, 00:13 and so on, every hour, every day to keep amboss informed, that your node is online. If it isn't, amboss will show your node as offline after 10 min.

## Make it seen publicly

Log in to amboss.space and check your monitoring settings. You'll find a switch to turn on your healthcheck settings for the public.
