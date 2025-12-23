# EasyCheatDetector (ECD) Analysis Dump via ida pro

# DO YOU WANT TO BYPASS? GO HTTPS://SHOP.CIAEY.CC



## 1. Reporting Infrastructure
*   **Endpoint:** `https://fungun.net/ecd/`
*   **Update URL:** `https://fungun.net/ecd/list/`
*   **Protocol:** HTTPS POST (JSON Payload)
*   **Key Fields:** `uuid`, `hostname`, `gamehash`, `jointime`, `mem_prot`, `hooked`, `hookedaddr`, `demopath`, `exception`, `report_id`.
*   **User-Agent:** User-Agent string likely mimics a standard browser or game client (needs dynamic capture to verify).

## 2. Signature Storage & Updates
*   **Primary Database:** `cheat.db` (Hosted on GitHub `UnrealKaraulov/UnrealCheatFinderDB`) or `signatures.xml` (Hosted on `fungun.net`).
*   **Format:** The `cheat.db` file appears to be a custom text-based format that is **reversed** and locally scrambled/obfuscated.
    *   **Header/Footer:** `]ET0_SCAN_DB` / `BD_NACS_0TE]` (Reversed).
    *   **Decoded Strings (Partial):**
        *   `FATAL ERROR UNREAL CHEAT FINDER IS DISABLED!`
        *   `PLEASE DOWNLOAD EASYCHEATDETECTOR`
        *   `fungun.net/ecd`
        *   `github.com/UnrealKaraulov/EasyCheatDetector`
*   **Encryption Key:** A 160-character master key was identified in the binary:
    `G7@qZ!xP3#tL9$wF8^mR2&jH6*eD1%vT4@kN5!sB0#yQ3^uX8$zC7&lM2*tJ9^oF1!pR6#hV4@wT5%jK8&nZ3*eD0^xL7!qP2#yF9$kN1^mR6&jH4*tB8@wC5%zX3^oL0!pT2#jK9&nF1*eD7^mR5@wQ8%lZ3#tJ0^xP6!kN2&yF4$hR1^jM9*tB5@wC8%zX3^oL0!pT2#jK9&nF1` (Approximate - see binary dump).

## 3. Embedded Resources & Databases
*   **Blacklisted Certificates (Resource 111):**
    *   **Format:** PKCS#12 (PFX)
    *   **Password:** `TestCertPassword555`
    *   **Functionality:** Loads this store to check against running process/driver signatures. If a match is found (e.g., a known cheat certificate), it flags the process.
    *   **Location:** Embedded in the binary, typically referenced around `0xAFE7E8` or loaded via `PFXImportCertStore`.
*   **SQLite Database:**
    *   **Location:** Embedded at offset `0xA6FDD8` (Header `SQLite format 3`).
    *   **Content:** Contains cached rules, configuration, and possibly static signatures that are not in the dynamic XML/DB.
    *   **Tables:** Likely contains tables for `scans`, `rules`, and `logs`.

## 4. Detection Logic
*   **API Hook Monitoring:**
    *   Monitors `NtQueryObject`, `NtTerminateThread`, `NtGetContextThread`, `Process32FirstW`, `Process32NextW`, `EnumProcesses`, `GetModuleBaseNameW`.
    *   Checks for `jmp` instructions or modifications in the prologue of these functions.
*   **Library Targeting:**
    *   Specific checks for `Blackbone` (memory manipulation library) and `asmjit`.
    *   Checks for "hacker" tools like `Cheat Engine`, `x64dbg`, `OllyDbg` via window class names and process names.
*   **HWID Collection:**
    *   Uses `MachineGuid` from Registry.
    *   Collects Disk Serial Numbers, CPU info (`CPUID`), and MAC addresses.
*   **License Check:**
    *   Requires user to agree to "User Agreement".
    *   Strings: `You have agreed to the user agreement`, `Click here to open Report`.
    *   If the "Confirm" button hangs, it may be due to:
        1.  **Anti-Debugging:** `IsDebuggerPresent` or `CheckRemoteDebuggerPresent` checks failing the license grant.
        2.  **Network Failure:** Inability to reach `fungun.net` to verify the session or download the latest `signatures.xml`.
        3.  **Database Corruption:** If the embedded SQLite DB fails to initialize (Header `0xA6FDD8`), the UI might freeze.


