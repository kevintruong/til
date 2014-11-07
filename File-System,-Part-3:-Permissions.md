# Under construction

# Under construction
# Under construction
# Under construction !!
## How can I have the same file appear in two different places in my file system?
(From code? Command?)
Reference counting?

## What happens when I `rm` (remove) a file?
rm = unlinking?

## How do I change the permissions on a file?
```
chmod 644 /bin/sandwhich
chmod 755 /bin/sandwhich
chmod ugo-w /bin/sandwhich
chmod o-rx /bin/sandwhich
```

## How do I read the permission string from ls?

## What is sudo?

## How do I change ownership of a file?

## How do I set permissions from code?

`chmod(const char *path, mode_t mode);`

## Why are some files 'setuid' what does this mean? ?
set-user-ID-on-execution/set-group-ID-on-execution
Why are they useful?

## What permissions does sudo run as ?
```
ls -l /usr/bin/sudo
-r-s--x--x  1 root  wheel  327920 Oct 24 09:04 /usr/bin/sudo
```
## What's the difference betweeen getuid() and geteuid()?
## How do I ensure only privileged users can run my code?
Use geteuid() == 0
