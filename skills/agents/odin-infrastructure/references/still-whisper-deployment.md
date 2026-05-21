# Still Whisper + Sidecar Deployment

## SSH Access
- Use key: `~/.ssh/id_ed25519_agent_tail8a8496_ts_net`
- Command pattern: `ssh -i ~/.ssh/id_ed25519_agent_tail8a8496_ts_net -o StrictHostKeyChecking=no root@100.94.237.15`

## Whisper Installation (faster-whisper)
```bash
apt-get update && apt-get install -y python3-pip ffmpeg
pip3 install faster-whisper --break-system-packages
python3 -c "
from faster_whisper import WhisperModel
model = WhisperModel('large-v3', device='cpu', compute_type='int8')
print('large-v3 ready')
"
```

## Sidecar on Still
- Sidecar already configured to use LM Studio endpoint:
  `LMSTUDIO_URL = "http://127.0.0.1:1234/v1/audio/transcriptions"`
- .env: `WHISPER_MODEL=large-v3`
- Service runs under `agent` user at `/home/agent/odin-sidecar/`

## LM Studio Status Check
```bash
curl -s http://localhost:1234/v1/models
ps aux | grep llmster
```

## Disk Cleanup Patterns (still)
- Common hogs: `/home/snorcup/.lmstudio` (models), `/home/odin/ollama-data`
- Safe sequence:
  1. `apt-get clean && apt-get autoremove -y`
  2. `journalctl --vacuum-time=3d --vacuum-size=50M`
  3. `docker system prune -af --volumes`
  4. Remove old models: `rm -rf /home/snorcup/.lmstudio/models/*`
  5. Remove ollama: `rm -rf /home/odin/ollama-data`

Current typical free space target after cleanup: ~2.4G on 31G root fs.