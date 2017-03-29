# Bash Toolbelt

> Note: Work in progress.

### Misc Helpers


```sh
function requireField () {
  if [ -s $2 ]; then
    printf "${red}${1} is a required parameter\n${yellow}${3}\n\n${reset}"
  fi
}
```

