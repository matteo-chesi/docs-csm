# Reset root Password on LiveCD

It may become desirable to clear the password on the LiveCD.

The root password is preserved within the COW partition at `cow:rw/etc/shadow`. This is the
modified copy of the `/etc/shadow` file used by the operating system.

If a site/user needs to reset/clear the password for `root`, they can mount their USB on another
machine and remove this file from the COW partition. When next booting from the USB it will
reinitialize to an empty password for `root`, and again at next login it will require the password
to be changed.

Clear the password (macOS or Linux):

```bash
mypc:~ > mount -vL cow /mnt
mypc:~ > sudo rm -fv /mnt/rw/etc/shadow
mypc:~ > umount -v /mnt
```
