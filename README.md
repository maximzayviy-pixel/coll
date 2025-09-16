# TURN (coturn) on Render for Mattermost Calls

This repo contains a ready-to-deploy Render Blueprint for **coturn** (TURN server).
It lets Mattermost Calls work reliably behind NATs and mobile networks.

---

## 0) Prereqs
- Mattermost must already be running and have:
  - `MM_SERVICESETTINGS_SITEURL=https://mattermost-9g8i.onrender.com`
  - DB connected and working
  - File uploads enabled (optional)
- You'll deploy this repo as a **Blueprint** (not a web service).

## 1) Deploy coturn
1. Push these files to a new GitHub repo.
2. In Render → **New → Blueprint** → paste your repo URL → Deploy.
3. Wait until the private service **coturn** becomes *Live*.
   Copy its **Service Address** — it will look like `coturn-xxxx:3478`.

## 2) (Recommended) Set EXTERNAL_IP
1. Find the public IP of the Render host for your coturn service.
   - Easiest: in any terminal run `nslookup <YOUR-COTURN-HOST>` (without the `:3478`).
   - Or use any DNS lookup site.
2. In Render → coturn → **Environment**, set `EXTERNAL_IP` to that IP.
3. **Manual Deploy** the coturn service.

## 3) Configure Mattermost Calls
1. Open: System Console → Plugins → **Calls**.
2. In **ICE Servers Configurations**, paste JSON from `calls_config.json`
   after replacing **YOUR-COTURN-HOST** with the **hostname** from coturn Service Address
   (e.g., `coturn-xxxx.onrender.com` or `coturn-xxxx` if inside private DNS).
3. Click **Save**.
4. Go to Plugins → **Plugin Management**: Disable → Enable **Mattermost Calls**.

### Example JSON (edit YOUR-COTURN-HOST)
```json
[
  {
    "urls": [
      "turn:YOUR-COTURN-HOST:3478?transport=udp",
      "turn:YOUR-COTURN-HOST:3478?transport=tcp"
    ],
    "username": "turnuser",
    "credential": "turnpass"
  },
  {
    "urls": [
      "stun:YOUR-COTURN-HOST:3478"
    ]
  }
]
```

## 4) Test
- Start a call from a channel. Test both participants on Wi‑Fi.
- Then test one participant from a mobile network — audio should connect via TURN.

## 5) Troubleshooting
- **Cannot connect to call**:
  - Ensure `MM_SERVICESETTINGS_SITEURL` is your HTTPS domain.
  - In Calls settings, `ICE Servers Configurations` must be a **valid JSON array**.
- **Only works on same network**:
  - Set `EXTERNAL_IP` in coturn and redeploy.
  - Keep both UDP and TCP :3478 open (we expose both in Blueprint).
- **DNS errors in Calls logs**:
  - Use the exact **coturn host** from Service Address.
- **Still flaky**:
  - Add a STUN entry alongside TURN:
    - `"urls": ["stun:stun.l.google.com:19302"]`

## 6) Security Notes
- Credentials are static (`TURN_USER`, `TURN_PASS`). Change them to strong values.
- You can rotate them at any time; update Calls JSON accordingly and Save.
- For advanced setups, coturn also supports time‑limited credentials (TURN REST API).
