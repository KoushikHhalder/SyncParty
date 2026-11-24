# SyncParty

Private listening party for up to 10 phones (you + 9 friends). Your phone is the host: you add songs, friends join with your approval, and everyone hears the same song in sync. Works on iPhone and Android, installs as an app (PWA). No accounts, no backend of your own.

## Put it online (one time, ~5 minutes, from your phone)
The app must be served over **https** (needed for WebRTC, install-as-app and screen wake lock).
Easiest options:
1. **Netlify Drop** – open app.netlify.com/drop, upload the `dist` folder (or this zip's contents). You get an https link.
2. **GitHub Pages** – create a repo, upload the files in `dist`, then Settings -> Pages -> deploy from `main` / root.
3. **Vercel / Cloudflare Pages** – import the repo, no build step, output directory = `dist`.
Send your friends that link once; after that you only share a party code or QR.

## Hosting a party (you)
1. Open the link, enter your name, tap **Host a party**, name it.
2. Share the code, link or QR. Tap **Open the party**.
3. Tap **Add songs** and pick audio files from your phone (mp3, m4a, wav...). Friends' phones download them automatically.
4. Friends appear under the people icon: **Approve** / **Reject**. You can Remove, Lock the party, or End it in Settings.
5. Press play. Keep your screen on and the app open: your phone is the DJ booth.

## Joining (friends)
Open the link (or enter the code), type a name, **Request to join**, wait for approval, tap once to enable sound.

## Fine-tune by ear (Settings -> Fine-tune this phone)
If one phone sounds slightly late or early (typical with Bluetooth speakers), slide *Audio delay compensation* on that phone until it lines up. It is saved on the phone.

## How sync works
Server-authoritative timeline: the host holds `{track, position, anchor time}`. Each phone measures its clock offset to the host (ping/pong, lowest-RTT sample), schedules playback for a future host timestamp, then every ~1.5 s compares its position to the timeline and fixes drift (tiny speed change under 400 ms, hard seek above). Songs are sent to every phone before play so playback never depends on live streaming.

Typical result: within 100-250 ms on normal Wi-Fi/4G. Bluetooth speakers add their own delay that no app can measure.

## Known limits (be aware)
- **Host phone must stay open.** If the host's screen locks or the browser is suspended, the party freezes; reopen the app, tap **Resume**, and re-add songs (files cannot be saved by a web page). Friends reconnect automatically without re-approval.
- **iPhone listeners:** audio can pause when the screen locks. The app reconnects and resyncs on return. Keep screens on for best results.
- **Connection:** uses the free public PeerJS signaling server plus Google/Twilio STUN. Most home Wi-Fi, hotspots and 4G work. Strict carrier NAT or corporate Wi-Fi may need a TURN server (not included).
- Only play music you own or are licensed to play. DRM streams (Spotify, YouTube) cannot and must not be captured.
- Master-only controls are enforced on the host: it ignores and logs any control message from a guest. This is a private peer-to-peer party, not a multi-tenant cloud service.

## Optional: your own signaling server
`npm i peer && npx peerjs --port 9000`, then open the app with `#ps=your-host:9000`.

## Develop / test
```
npm install
node --test test/core.test.js   # 28 tests: sync math, permissions, capacity, 10 simulated users, reconnects
node build.js                   # rebuilds dist/index.html (single file)
```
