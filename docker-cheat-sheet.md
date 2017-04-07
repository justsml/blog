### Security

https://github.com/docker/docker-bench-security

https://nodesecurity.io/ (not docker - but important for node apps)


### Remove Old Images: 

```sh
docker system prune
# AND/OR
docker image prune

# OLD: 
docker rmi $(docker images -q -f dangling=true)

```

