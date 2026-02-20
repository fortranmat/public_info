# VS Code Server Setup on Contabo VPS (Ubuntu 24.04)

## Prerequisites
- Contabo VPS with Ubuntu 24.04 LTS
- SSH access (Putty or terminal)

---

## Step 1 — Install code-server

```bash
curl -fsSL https://code-server.dev/install.sh | sh
```

---

## Step 2 — Enable and Start

```bash
sudo systemctl enable --now code-server@root
```

---

## Step 3 — Get Password

```bash
cat ~/.config/code-server/config.yaml
```

Note down the password from the output.

---

## Step 4 — Fix Bind Address (Important!)

By default code-server binds to `127.0.0.1` (localhost only) which blocks external access.
Change it to `0.0.0.0` to allow browser access:

```bash
sed -i 's/127.0.0.1:8080/0.0.0.0:8080/' ~/.config/code-server/config.yaml
```

Verify the change:
```bash
cat ~/.config/code-server/config.yaml
```

Expected output:
```yaml
bind-addr: 0.0.0.0:8080
auth: password
password: your-password-here
cert: false
```

---

## Step 5 — Allow Firewall Port

```bash
sudo ufw allow 8080
```

> ⚠️ If ufw is not installed:
> ```bash
> sudo apt install ufw -y
> sudo ufw allow 22    # SSH - always allow this first!
> sudo ufw allow 80    # HTTP
> sudo ufw allow 8080  # VS Code
> sudo ufw allow 10000 # Webmin
> sudo ufw enable
> ```

---

## Step 6 — Restart code-server

```bash
sudo systemctl restart code-server@root
```

---

## Step 7 — Access in Browser

```
http://YOUR_VPS_IP:8080
```

Enter the password from Step 3.

---

## Change Password (Recommended)

> 💡 You don't need Putty/SSH to do this! Use the **built-in terminal inside VS Code browser**:
> `VS Code (browser) → Terminal menu → New Terminal`
> Then run the commands below directly from your browser.

```bash
sed -i 's/password: .*/password: yournewpassword/' ~/.config/code-server/config.yaml
sudo systemctl restart code-server@root
```

Replace `yournewpassword` with your chosen password.

Verify the change:
```bash
cat ~/.config/code-server/config.yaml
```

Then logout and login again with the new password at `http://YOUR_VPS_IP:8080`.

---

## Useful Commands

| Command | Purpose |
|---|---|
| `sudo systemctl status code-server@root` | Check if running |
| `sudo systemctl restart code-server@root` | Restart |
| `sudo systemctl stop code-server@root` | Stop |
| `cat ~/.config/code-server/config.yaml` | View config |
| `ss -tlnp | grep 8080` | Check port listening |

---

## Troubleshooting

| Problem | Solution |
|---|---|
| Port not opening | Check bind-addr is `0.0.0.0:8080` not `127.0.0.1:8080` |
| dpkg lock error | Wait for background apt process to finish, then retry |
| ufw not found | `sudo apt install ufw -y` |
| Page not loading | Check Contabo firewall in control panel allows port 8080 |

---

## Firewall Troubleshooting (Virtualmin/firewalld)

If code-server is running and port is correct but browser still won't load, the issue is likely **Virtualmin's firewalld** blocking the port (separate from ufw).

**Step 1 — Check which ports are allowed:**
```bash
sudo firewall-cmd --list-ports
```

Example output (port 8080 missing = blocked):
```
20/tcp 2222/tcp 10000-10100/tcp 20000/tcp 49152-65535/tcp
```

**Step 2 — Add port 8080:**
```bash
sudo firewall-cmd --add-port=8080/tcp --permanent
sudo firewall-cmd --reload
```

**Step 3 — Verify:**
```bash
sudo firewall-cmd --list-ports
```

Port 8080 should now appear in the list and browser access will work.

> ⚠️ If you installed Webmin/Virtualmin via Contabo OS reinstall, `firewalld` replaces `ufw` as the active firewall. Always use `firewall-cmd` instead of `ufw` in this case.

---

## Notes
- code-server version installed: v4.109.2
- Default port: 8080
- Config file: `~/.config/code-server/config.yaml`
- Runs as root user on this setup
