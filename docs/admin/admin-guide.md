# Administrator Guide

This document is a guide only for the administrator. Normal users without the administrator's right have nothing to see here.

## Adding user

Suppose that the new user's ID is `alice` and its real name is _Alice Liddell_.

``` no-highlight
sudo wwuseradd alice
sudo passwd alice                      # set the password.
sudo usermod -c 'Alice Liddell' alice  # this is not mandatory,
                                       # but would be useful to identify the user.
```

If `alice` would be added to the group `usercl1`,

``` no-highlight
sudo usermod -aG usercl1 alice
```

Now check `/etc/passwd` and `/etc/group`:

``` no-highlight
grep alice /etc/passwd /etc/group
```

If everything is fine, synchronize the files to compute nodes.

``` no-highlight
sudo wwsh file resync
```

The synchronization might take several minutes. To check the status, run

``` no-highlight
sudo pdsh -w 'compute-0-[0-22]' grep alice /etc/passwd
```

A useful command to force the synchronization is

``` no-highlight
sudo pdsh -w 'compute-0-[0-22]' 'rm /tmp/.wwgetfiles_timestamp; SLEEPTIME=1 /warewulf/bin/wwgetfiles'
```
