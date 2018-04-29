+++
draft = false
date = "2018-04-28T15:27:34+01:00"
title = "GPG Basics"
categories = ["Linux","Tooling"]

+++

Generate a strong 4096 bit key
```
gpg --full-generate-key
```

List generated keys
```
gpg --list-secret-keys
```

Upload key to default servers
```
gpg --send-keys <your_key_uid>
```

Specify a specific remote server
```
gpg --keyserver hkps://keyserver.ubuntu.com --send-keys <your_key_uid>
```

Exporting your key for orther devices
```
gpg --output sec_key.gpg --armor --export-secret-key <your_key_uid>
```
