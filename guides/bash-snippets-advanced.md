# Bash Toolbelt

> Note: Work in progress.

### Misc Helpers


```sh
function requireField () {
  if [ 1 -lt $(echo "$2" | wc -c) ]; then
    printf "${red}${1} is a required parameter\n${yellow}${3}\n\n${reset}"
    exit 123
  fi
}
# Usage: 
requireField  "PASSWORD"  $PASSWORD   "ERROR: PASSWORD is required environment var."
# 3 Parameter: ^Name^     ^Variable^          ^Err Message^
```

![image](https://cloud.githubusercontent.com/assets/397632/24480679/e51db4bc-14a2-11e7-9aad-3b02eb813df5.png)
