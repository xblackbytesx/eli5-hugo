+++
date = "2017-03-28T13:55:27+02:00"
lastMod = "2018-08-23T00:10:12+01:00"
title = "[OUTDATED] ghost blog n00b guide"
draft = false

+++

# Basic setup
`curl -L https://ghost.org/zip/ghost-latest.zip -o ghost.zip`  

`unzip -uo ghost.zip -d ghost`

`cd ghost && npm install --production`

# Running on production

### Install PM2 server
`sudo npm install pm2 -g`

### Configure the app
In your ghost folder create a config.json file with the following content:
```
{
  "apps": [
    {
      "name": "ghost",
      "script": "npm",
      "args": "start --production"
    }
  ]
}
```

### Start the server
`pm2 start config.json`
