### there's a jail.zsh file given:

```
#!/bin/zsh
print -n -P "%F{green}Specify your charset: %f"
read -r charset
# get uniq characters in charset
charset=("${(us..)charset}")
banned=('*' '?' '`')

if [[ ${#charset} -gt 6 || ${#charset:|banned} -ne ${#charset} ]]; then
    print -P "\n%F{red}That's too easy. Sorry.%f\n"
    exit 1
fi
print -P "\n%F{green}OK! Got $charset.%f"
charset+=($'\n')

# start jail via coproc
coproc zsh -s
exec 3>&p 4<&p

# read chars from fd 4 (jail stdout), print to stdout
while IFS= read -u4 -r -k1 char; do
    print -u1 -n -- "$char"
done &
# read chars from stdin, send to jail stdin if valid
while IFS= read -u0 -r -k1 char; do
    if [[ ! ${#char:|charset} -eq 0 ]]; then
        print -P "\n%F{red}Nope.%f\n"
        exit 1
    fi
    # send to fd 3 (jail stdin)
    print -u3 -n -- "$char"
done
```

in this we have to give 6 chars that we will use and some special characters are banned. 

i tried running find * as it has 6 characters
this recursively searched all directories and gave:

![image](https://github.com/oxo-crab/diceCTF/blob/main/image.png)

we need more than 6 characters to access it.

Tried killing the job which zsh script forced to remove the check but that didn't work, ultimately decided to turn to zsh man page, found that zsh automatically changes the directory
without using cd, and can perform automatic file expansion while changing directory so need to type the full name, but that didn't work.
Found about the use of ` `!:^` charcters to repreat previous commands in history but that didn't work to.

After viewing other writeups, i came to know we had to use [] and ~~
[] matches any character inside the braces and by putting ~~ in [] it like this `[~~]` it will match with any character not given in the set, so we havd to just do 
`[~~][~~][~~]/[~~][~~][~~]/[~~][~~][~~][~~][~~][~~][~~][~~]/[~~][~~][~~][~~]/[~~][~~][~~][~~][~~][~~][~~]` and it would run getflag 

Also went through this [video](https://www.youtube.com/watch?v=poHirez8jk4)(ddexec/ecverythingexec) which is about bypassing the read only enviornment to execute any binary you want to.










