# SSH Key Authentication for Termux on Android

## On the control host (agent)

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_odin_pixel -N "" -C "odin-to-pixel"
```

## On the Android device (Termux)

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh

cat >> ~/.ssh/authorized_keys << 'KEY'
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBdXpOzkNg0iOTfoIiOhFOho1UgH1AKv2yEUwlMF97fE odin-agent-to-pixel9pro
KEY

chmod 600 ~/.ssh/authorized_keys
```

## Connection command

```bash
ssh -i ~/.ssh/id_odin_pixel -p 8022 u0_a318@100.86.74.48
```

## Boot script for persistent SSH

Create `~/.termux/boot/start-sshd.sh`:

```bash
#!/data/data/com.termux/files/usr/bin/sh
sshd
```

Make executable:

```bash
chmod +x ~/.termux/boot/start-sshd.sh
```

Enable "Run boot script" in Termux Settings → Boot.