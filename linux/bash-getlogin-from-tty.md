# Get login user from tty in shell script bash/sh (emulate getlogin(3))

(The following can easily be circumvented (like getlogin(3).
DO NOT USE FOR SECURITY-RELATED PURPOSES!)

Say you want to know the login user from a bash script (as in even
if you have run sudo you still want to get the original user name).

There's a libc function that does this called getlogin(3). It looks
at the owner of your terminal to try to figure out who the login user is.

We can emulate that behavior in shell (on linux):

```
username=$(stat -c '%U' $(tty))
```
