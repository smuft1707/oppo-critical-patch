# PoC: Remote Silent Install via Insecure Deserialization (Oppo App Market)

## ⚠️ Disclaimer
This repository is created for security research purposes only and is part of a bug report submitted to the Oppo Security Response Center (OSRC) via HackerOne. Unauthorized use of this material is strictly prohibited.

## 📝 Vulnerability Overview
- **Target App:** `com.heytap.market` (Oppo App Market)
- **Vulnerability:** Insecure Deserialization in `OapsProvider` / `WebBridgeActivity`
- **Vector:** Remote via Deep Link (`oaps://`)
- **Impact:** Critical (Full Device Compromise via Silent APK Installation)
- **CVSS Score:** ---

## 🔍 Technical Description
The Oppo App Market application exports an activity named `com.cdo.oaps.host.old.WebBridgeActivity`. This activity processes incoming intents through the `oaps://` scheme. 

The parameter `cb` within the intent is passed directly to a deserialization sink without proper validation. Since `com.heytap.market` is a system-privileged application holding the `INSTALL_PACKAGES` permission, an attacker can craft a serialized object (Gadget Chain) to invoke the internal `PackageInstaller` or `IPackageManager` to install an APK file from the local storage.

## 🚀 Attack Flow (Zero-Interaction)
1. **Delivery:** Attacker sends a malicious URL to the victim.
2. **Auto-Download:** Upon opening the URL, the `index.html` triggers an automatic download of `exploit.apk` to the `/sdcard/Download/` directory using JavaScript.
3. **Exploit Trigger:** After a short delay, the page automatically redirects to:
   `oaps://add?ckey=exploit&cb=[PAYLOAD]`
4. **Escalation:** The App Market receives the intent, deserializes the malicious payload, and executes the installation command in the background.
5. **Outcome:** The rogue application is installed and launched without any user prompts or "Unknown Sources" warnings.

## 📂 Repository Contents
- `index.html`: The malicious landing page featuring auto-download and auto-intent trigger.
- `exploit.apk`: A dummy Android application used to demonstrate a successful installation.

## 🎥 Reproduction Steps
1. Open the following URL on an Oppo device: `https://smuft1707.github.io/oppo-critical-patch/`
2. Observe the automatic download of the APK.
3. Observe the redirection to the App Market.
4. Verify that `com.poc.silentinstall` (or the dummy app) is installed on the system.

---
Recommendation: > "The application should avoid using ObjectInputStream.readObject() on data coming from untrusted Intents. Instead, use a safer serialization format like JSON or implement a strict ObjectInputFilter to whitelist only expected classes during deserialization."
---
**Author:** [smuft1707](https://github.com/smuft1707)
**Reported to:** Oppo Security Team
