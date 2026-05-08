---
pubDatetime: 2022-09-23T04:58:53Z
modDatetime: 2026-05-08T12:07:00.000Z
title: How to host website from android
slug: how-to-host-websites-from-your-android
featured: true
draft: false
tags:
  - configuration
  - docs
  - termux
  - pm2
description: A detailed guide on hosting lightweight websites from an Android device using Termux, PM2, and Cloudflare.
---

## Table of contents

## Why host on Android?

Honestly, there is no strong reason to do this — and I want to be upfront about that. It is not a one-size-fits-all solution. Docker does not run natively on Android at all, so you are limited to lighter runtimes like Node.js, and by extension, frameworks like Astro, Next.js, or plain static sites.

I had a spare Android lying around and felt like putting it to good use while learning Linux in the process. I primarily use Windows for its game compatibility, but most servers run Linux — and Android itself is Linux-based (not to be confused with distros like Pop!_OS or One UI, which ship on other devices). So this was a good excuse to get familiar with the terminal.

<details>
<summary>NOTE</summary>

I am writing this blog with some config already done on my Android, so I might forget some steps. Additionally, not all Android OSes are the same, so you may run into additional hurdles. If that happens, feel free to reach out.

</details>

The main challenge is that Android is designed to save battery by aggressively killing background processes. To turn your phone into a 24/7 server, you have to fight the OS a little.

## Initializing Termux

Termux is a terminal emulator that gives you a full Linux environment on Android. This is where all commands will run.

1. **Download from F-Droid:** Always use the F-Droid version. The Play Store version is outdated and will throw "repository under maintenance" errors when installing packages.This is what the community suggests (from reddit) but i have been using one from playstore n have not run into any issues.

2. **Disable Battery Optimization:** Go to your phone's app settings for Termux and set battery usage to **"Unrestricted"**. This is step one of keeping it alive.

3. **Enable Developer Options and disable the Phantom Process Killer:** On Android 12+, the OS quietly kills background apps that spawn too many child processes — Node.js and PM2 are notorious triggers for this.
    - Go to **Settings → Developer Options**.
    - Find **"Disable child process restriction"** and toggle it **ON**.
    - This is the most common reason servers crash on modern Android, even with everything else configured correctly.

4. **Acquire a Wakelock:** Run `termux-wake-lock` in your terminal, or swipe down the notification shade and tap "Acquire wakelock." This prevents the CPU from entering deep sleep when the screen is off, which would otherwise lag or kill your server.

5. **Lock Termux in Recents:** When you open the recent apps view and long-press Termux, you should see a lock icon. Enable it. This prevents the "Clear All" button from accidentally killing your session.

### System Setup

Once inside Termux, update the repositories and install the tools:

```bash
pkg update && pkg upgrade
pkg install nodejs-lts pnpm git cloudflared openssh procps
npm install -g pm2
```

> **Why pnpm?** pnpm is faster and more disk-efficient than npm, which matters on mobile hardware where I/O can be a bottleneck.

## Setting up SSH

Typing on a phone screen is painful. SSH lets you control the device from your laptop using a proper keyboard.

Termux uses port **8022** instead of the standard port 22, because Android doesn't grant it root access to bind to privileged ports.

1. **Start the server:** Run `sshd`.
2. **Find your phone's local IP:** Run `ifconfig | grep inet` and look for the `wlan0` entry. It will look like `192.168.x.x`.
3. **Connect from your PC:**
```bash
ssh <any-username>@<phone-ip> -p 8022
```

Termux accepts any username — it only checks the password. On first connection, type `yes` (the full word) when prompted about the fingerprint.

> **Host Key Verification Failed?** This happens when your phone's IP changes between sessions. Your PC remembers the old IP and flags the new one as suspicious. Fix it by running this on your laptop:
> ```bash
> ssh-keygen -R [<old-phone-ip>]:8022
> ```
> Then reconnect normally.

Ideally you should just go into your router's admin page login with appropriate id password (usually written on back of router) n assign it static DHCP reservation , different router call these by different names , you just want to tell your router that the device with "this mac id" needs to be given "192.168.1.10" ip. This would also require you to disable MAC randomization , Most devices have these by default for privacy n security reasons.

## Process Management with PM2

PM2 (Process Manager 2) keeps your website running in the background and automatically restarts it after a crash. Think of it as a restaurant manager — you tell it what to run, then you can walk away.

### 1. Launching the Site

Navigate to your project folder and start your site with pnpm. The key here is using `--` twice: once to separate PM2's arguments from pnpm's, and once to pass flags through to the underlying Astro dev server.

```bash
cd ~/Blog
pm2 start pnpm --name "KodiBlog" -- run dev -- --host --port 4321
```

The `--host` flag is required. Without it, the server only listens on `localhost` — the Cloudflare tunnel cannot reach it.

> **Using `preview` instead of `dev`?** If you have already built your site with `pnpm build`, you can serve the production build:
> ```bash
> pm2 start pnpm --name "KodiBlog" -- preview --host 0.0.0.0 --port 4321
> ```

### 2. Launching the Tunnel

Manage your Cloudflare tunnel through PM2 as well:

```bash
pm2 start "cloudflared tunnel run <your-tunnel-name>" --name "cf-tunnel"
```
In my case you can get away with not pssing `<your-tunnel-name>` since ther is only one tunnel , this is a best practice , have only one tunnel you can serve multiple websites through this single tunnel.

### 3. Saving the State

After starting your processes, save the list:

```bash
pm2 save
```

And add this line to your `~/.bashrc` so PM2 auto-revives your processes every time you open Termux:

```bash
echo "pm2 resurrect" >> ~/.bashrc
```

If you ever restart your phone and find an empty PM2 list, just run `pm2 resurrect` manually to bring everything back.

## Setting up Cloudflare Tunnel

Cloudflare Tunnels connect your local server to a public domain without port forwarding or a static IP from your ISP. Your router stays untouched.

sadly you would need a domain for this to work , cloudflare does provide domains but they are dynamic n change regularly.You can rent domains from any registrars .(GoDaddy , hostinger are good ones , additional steps would be required to move your domain from where you bought to Cloudflare's DNS ,)

1. **Login:** Run `cloudflared tunnel login` and open the URL it gives you.
2. **Create the Tunnel:** Run `cloudflared tunnel create my-website`.
3. **Configure:** Create `~/.cloudflared/config.yml`:

```yaml
tunnel: <your-tunnel-id>
credentials-file: /data/data/com.termux/files/home/.cloudflared/<tunnel-id>.json

ingress:
  - hostname: yourdomain.com
    service: http://localhost:4321
  - service: http_status:404
```

4. **Route DNS:** Run `cloudflared tunnel route dns my-website yourdomain.com`.

### Running Multiple Sites

The same tunnel can serve multiple domains simultaneously — you do not need a separate tunnel per site. Just add additional entries to `config.yml` (each site on a different port) and restart the tunnel:

```yaml
ingress:
  - hostname: blog.yourdomain.com
    service: http://localhost:4321
  - hostname: links.yourdomain.com
    service: http://localhost:4322
  - service: http_status:404
```

Then run `cloudflared tunnel route dns my-website links.yourdomain.com` for the new domain and `pm2 restart cf-tunnel` to reload the config.

## Updating the Website

When you push new code to GitHub, pull and restart:

```bash
cd ~/Blog
git pull
pnpm install
pnpm build
pm2 restart KodiBlog
```

## Troubleshooting

**Website stops randomly:** Double-check that "Disable child process restriction" is toggled on in Developer Options. This is the most common cause of crashes on Android 12+.

**Port already in use:** If a process did not shut down cleanly, find and kill it:
```bash
lsof -i :4321
kill <PID>
```

**SSH "Host Key Verification Failed":** Your phone's IP changed. On your laptop, run:
```bash
ssh-keygen -R [<old-ip>]:8022
```

**Monitoring:** Use `pm2 monit` for a real-time dashboard of CPU and RAM usage, or `pm2 logs KodiBlog` to tail the server output.

---

Hosting on Android is surprisingly stable once the OS-level restrictions are handled. It's a free, low-power way to host personal projects — and honestly, it's a better Linux crash course than any cloud tutorial.