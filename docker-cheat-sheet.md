
### Remove Unaffiliated Images: (skips running)
```sh
docker rmi $(docker images -f dangling=true -q)
```

