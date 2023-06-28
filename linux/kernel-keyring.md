# Kernel keyring Notes

Sometimes you want a secret to be available to read by say a single
terminal session, but you'd like more protection than just sticking that
secret in an environment variable.

One option here is to use the kernel keyring facility to create a new session
keyring in a single terminal:

```
# create a new session keyring:
keyctl session -

# add a key to the keyring
pass some/secret | head -1 | keyctl padd user my_key_name @s

# show current keyring
keyctl show

# use keyctl as password for some process:
export FOOBAR_PASSWORD_COMMAND="keyctl pipe $(keyctl search @s user my_key_name)"
```



References:
- https://blog.cloudflare.com/the-linux-kernel-key-retention-service-and-why-you-should-use-it-in-your-next-application/
- https://mjg59.dreamwidth.org/37333.html
