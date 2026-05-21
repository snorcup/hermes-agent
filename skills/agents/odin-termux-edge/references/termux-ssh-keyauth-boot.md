# Termux SSH Key Auth + Boot Persistence (Pixel Audio Capture Device)

## When to Use
Setting up reliable, passwordless remote access to the Termux device running the Odin audio capture client (Pixel 9 Pro). This enables the agent to inspect logs, restart capture, and manage the client side without repeated password prompts.

## Setup Steps

### 1. Generate dedicated key on agent
```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_pixel -N "" -C "agent-to-pixel-termux"
```

### 2. Copy key to Pixel (one-time with password)
```bash
sshpass -p 'sshd' ssh-copy-id -i ~/.ssh/id_pixel.pub -p 8022 \
  -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null \
  u0_a320@100.86.74.48
```

### 3. Create Termux:Boot script on Pixel
```bash
ssh -i ~/.ssh/id_pixel -p 8022 u0_a320@100.86.74.48 '
mkdir -p ~/.termux/boot
cat > ~/.termux/boot/start-sshd.sh << "EOF"
#!/data/data/com.termux/files/usr/bin/bash
sshd
echo "sshd started at $(date)" >> ~/.termux/boot/sshd.log
EOF
chmod +x ~/.termux/boot/start-sshd.sh
'
```

### 4. Verify
```bash
ssh -i ~/.ssh/id_pixel -p 8022 u0_a320@100.86.74.48 'echo "Key auth OK"'
```

## Notes
- Username on Termux is typically `u0_a320` (or similar `u0_aXXX`).
- Termux:Boot package must already be installed.
- The boot script starts `sshd` on every Termux launch / device boot.
- Use `-o UserKnownHostsFile=/dev/null` during initial setup to avoid host key conflicts after Termux restarts.