### Overview
A coordinated hacktivist operation targeting a newspaper. Two employees (Sonia Gose, Senior Editor and Ronnie McLovin, Editorial Intern) were separately phished via fake recruiter emails. The attacker planted backdoors, established persistent RDP tunnels via Plink, and ultimately injected a fabricated news article into the print workflow - which was then sent to the newspaper's printer (Clark Kent) for publication.

### Attack Timeline

```
Jan 2, 2024    → Ronnie McLovin hired (Editorial Intern)
Jan 5, 2024    → Sonia Gose phished via newspaper_jobs@gmail.com
Jan 5, 10:23   → Sonia clicks link; Valdorian_Times_Editorial_Offer_Letter.docx downloaded
Jan 5, 10:24   → hacktivist_manifesto.ps1 executed on UL0M-MACHINE
Jan 5, 11:22   → Scheduled task created (every 5 hours persistence)
Jan 6, 02:39   → Plink RDP tunnel established → C2: 136.130.190.181
Jan 10, 2024   → Ronnie McLovin phished via valdorias_best_recruiter@gmail.com
Jan 10, 08:55  → Editorial_J0b_Openings_2024.docx downloaded (A37A-DESKTOP)
Jan 10, 08:55  → hacktivist_manifesto.ps1 dropped on Ronnie's machine
Jan 11, 03:08  → Plink RDP tunnel established → C2: 168.57.191.100
Jan 31, 09:47  → Ronnie downloads fakestory.docx from hire-recruit.org
Jan 31, 10:26  → fakestory.docx renamed → OpEdFinal_to_print.docx
Jan 31, 11:11  → Ronnie (compromised) emails Clark Kent with fake article for publication
Jan 31, 11:49  → Memes exfiltrated: DankMemes.7z (password: thruthW!llS3tUfree)
Feb 1, 02:14   → 7z archives uploaded to hirejob.com/exfil_processor/upload.php
```

### Attack Chain

```
Fake recruiter emails (newspaper_jobs@gmail.com / valdorias_best_recruiter@gmail.com)
        ↓
Malicious .docx delivered from promotionrecruit.com / promotionrecruit.org
        ↓
WINWORD.EXE spawns: hacktivist_manifesto.ps1
        ↓  
Persistence: schtasks — "Hacktivist Manifesto" runs every 5 hours (bypass ExecutionPolicy)
        ↓
Plink.exe — SSH reverse tunnel to C2 (RDP over port 3389)
  → Sonia's machine → 136.130.190.181
  → Ronnie's machine → 168.57.191.100
  (Shared password: thruthW!llS3tUfree)
        ↓
Ronnie's account used to:
  1. Download fakestory.docx (hire-recruit.org / dark web: hirerecruit.com)
  2. Rename → OpEdFinal_to_print.docx
  3. Email Clark Kent (printer) to publish in tomorrow's paper
        ↓
Data exfil: DankMemes.7z → hirejob.com/exfil_processor/upload.php
```

### Indicators of Compromise (IOCs)

| Type | Value |
|------|-------|
| Phishing Senders | `newspaper_jobs@gmail.com`, `valdorias_best_recruiter@gmail.com` |
| Malicious Domains | `promotionrecruit.com`, `promotionrecruit.org`, `hire-recruit.org`, `hirejob.com`, `hirerecruit.com` (dark web) |
| C2 IPs | `136.130.190.181` (Sonia), `168.57.191.100` (Ronnie) |
| Backdoor Script | `hacktivist_manifesto.ps1` |
| Tool | `plink.exe` (SSH tunnelling) |
| Shared Password | `thruthW!llS3tUfree` |
| Fake Article | `fakestory.docx` → renamed to `OpEdFinal_to_print.docx` |
| Compromised Accounts | `sogose` (Sonia Gose), `romclovin` (Ronnie McLovin) |
| File Hashes (SHA256) | `60b854332e393a6a2f0015383969c3ac705126a6b7829b762057a3994967a61f` (Sonia's docx) |
| | `fa092856280dbddbc08d48ca996ac58eda22f1db978960c9c92ad0bd13ee197e` (Ronnie's docx) |
| | `1c3ef0407d5714037504c52f7abfa86c081fd7a021b52e2abe8a669f92413252` (hacktivist_manifesto.ps1) |

### Tools & Techniques (MITRE ATT&CK)

| Technique | ID | Tool / Method |
|-----------|-----|---------------|
| Phishing: Spearphishing Attachment | T1566.001 | Fake recruiter .docx lures |
| Malicious Macro / Script Execution | T1059.001 | hacktivist_manifesto.ps1 |
| Scheduled Task Persistence | T1053.005 | schtasks — every 5 hours |
| Protocol Tunnelling (RDP/SSH) | T1572 | plink.exe reverse tunnel |
| Masquerading | T1036 | fakestory.docx → OpEdFinal_to_print.docx |
| Exfiltration via Web Upload | T1567 | curl → hirejob.com |
| Influence Operation | - | Fake article injection into print workflow |

---

## 🛠️ Tools & Skills Demonstrated

| Tool / Platform | Usage Across Cases |
|---|---|
| **SIEM Log Analysis** | Process chain reconstruction, parent-child process tracing |
| **Splunk / ELK** | Timeline correlation, IOC hunting, lateral movement detection |
| **VirusTotal / Any.run** | Hash and file reputation analysis |
| **MITRE ATT&CK Framework** | TTP mapping for all three threat actors |
| **Wireshark / PCAP mindset** | Network traffic analysis (FTP exfil, C2 beaconing) |
| **Incident Response Playbooks** | Structured triage from alert → containment → remediation |

---

## 🧠 Key Lessons for SOC Analysts

1. **Monitor parent-child process chains** - `WINWORD.EXE` spawning `.ps1` files is a critical red flag.
2. **Audit scheduled tasks** - persistence mechanisms are often planted within minutes of initial access.
3. **Double-extension files are never legitimate** - `file.docx.exe` is always malicious.
4. **Base64 PowerShell = immediate escalation** - decode and analyse before anything else.
5. **Privileged account reuse** - one stolen domain admin credential (lihenry_domain_admin) encrypted 306 machines.
6. **Insider threats don't look like attacks** - Jane Smith's daily FTP job ran for 27 days undetected.
7. **Plink.exe on an endpoint = active reverse tunnel** - should never appear in a standard enterprise environment.
8. **Phishing lures are context-aware** - all three cases used tailored, believable pretexts (job offers, promo codes, healthcare tools).

---

## 📚 References & Frameworks

- [MITRE ATT&CK Enterprise Matrix](https://attack.mitre.org/matrices/enterprise/)
- [KC7 Cybersecurity Platform](https://kc7cyber.com)
- [Splunk Security Essentials](https://splunkbase.splunk.com/app/3435)
- [NIST Incident Response Guide (SP 800-61)](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)
