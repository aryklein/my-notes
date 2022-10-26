# How to setup a Yubikey in Arch Linux

To use Yubico Authenticator app in Linux you only need to install `ccid` package
and start the `pcscd` service.

The package `ccid` provides a generic USB interface driver for smart card reader.

```sh
sudo pacman -S ccid
sudo systemctl start pcscd.service
sudo systemctl enable pcscd.service
```

For 
Install the following management tools:

```bash
sudo pacman -Ss yubikey-manager yubikey-manager-qt
```
