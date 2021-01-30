# MongoDB

[manual](https://docs.mongodb.com/manual/)

## ENV

```sh
brew tap mongodb/brew
brew install mongodb-community


To have launchd start mongodb/brew/mongodb-community now and restart at login:
  brew services start mongodb/brew/mongodb-community
Or, if you don't want/need a background service you can just run:
  mongod --config /usr/local/etc/mongod.conf


brew services start mongodb/brew/mongodb-community
brew services list
```

- 安裝的東西包括
  - The mongod server
  - The mongos sharded cluster query router
  - The mongo shell

```sh
## 測試
mongo
mongotop
```

