# HP LaserJet Pro MFP M428fdw — Firmware Downgrade (restore 3rd-party cartridge support)

> ## 🙏 Credit — this solution belongs to the community
>
> **This fix exists because of [gointern](https://www.reddit.com/user/gointern/)**, who on
> r/printers in October 2022 tracked down the pre-block firmware, worked out the USB-stick
> method, and published it for everyone:
> **[“HP M428fdw M428dw firmware downgrade”](https://www.reddit.com/r/printers/comments/yd024g/hp_m428fdw_m428dw_firmware_downgrade/)**.
> Without that post, none of this would have been possible.
>
> Also thank you to **HuckleberryTypical59**, who re-hosted the firmware in 2024 when the
> original links started dying (keeping the fix alive through 2026), and to the thread's
> contributors who hardened the procedure — **CraftyLaw5921** (FAT32 + details),
> **John77379** (*Allow Updates* must be ON), and **mirfananwaar** (cold-reset trick).
>
> In gointern's own words: *"Hopefully this will be useful for somebody instead of paying
> over 200$ a cartridge to HP."* — Mission accomplished.

**Symptom:** HP Dynamic Security firmware (2025/2026 builds, e.g. `N002.2626A.00`) blocks
third-party HP 58A/58X (CF258A/X) cartridges with error **10.00.46** — *"cartridges have been
blocked by the printer firmware because they contain a non-HP chip"* — even with Cartridge
Policy = Off.

**Fix:** downgrade to HP's own signed firmware **`002.2247A`** (built 2022-11-14), which accepts
modern third-party chips. Confirmed working on **M428fdw** (and M329/M428/M429 family).

> Note: this file has circulated in community threads mislabeled as "1924A". The actual image
> inside is `002.2247A` — verified by filename, size, and the `@PJL COMMENT VERSION=` header.

## File

| File | Size | SHA-256 |
|---|---|---|
| `teton_dist_002.2247A_nonassert_appsigned_lbi_rootfs_secure_signed.ful2` | 93,234,933 B (88.92 MiB) | `a53e68772c3e7130265153b67fcb5f504757ef0ca093e39186b53ffc691e7c74` |

Genuine HP image extracted from HP's official "HP Firmware Updater" utility for the
M329/M428/M429 series (identical bytes in the Windows `.exe` and macOS `.dmg` packages).

## Procedure (USB stick — works even when network update channels are locked)

1. **Clear the current error first:** reinstall a genuine HP cartridge (even an empty one) until
   the printer reaches **Ready**. The update is silently discarded while an error is displayed.
2. Printer panel: **Setup → Printer Maintenance → Update the Printer → Manage Updates** →
   set **Allow Downgrade = Yes** and **Allow Updates = Yes**.
3. Format a USB stick (≥1 GB, **≤32 GB recommended**) as **FAT32**
   (macOS Disk Utility: Format *MS-DOS (FAT)*, Scheme *Master Boot Record*).
4. Copy the `.ful2` file to the stick (keep the filename).
5. Insert into the printer's **front** USB host port (the rear USB port will NOT work).
6. The panel prompts for a firmware update → confirm. Watch for **"Programming…"**, then the
   printer reboots (~5–10 min). **Never power-cycle during Programming.**
7. Verify: the EWS (http://printer-ip → device info) or the panel should show
   `TETONXXXXN002.2247A.00`.
8. **Lock it down:** Manage Updates → **Check Automatically = Off**, **Allow Updates = No**.
9. Reinstall the third-party 58X → prints.
   **Note:** on the first install you'll get a **"cartridges are not genuine HP" error message** —
   this is expected after the downgrade. The printer still flags non-HP chips as not genuine, but
   it *allows* them: dismiss/acknowledge the message (OK) and printing works normally. This is not
   the old `10.00.46` block — the cartridge stays usable.

## Troubleshooting

- **"Not genuine HP cartridge" message after the downgrade — normal.** The printer warns but
  lets you proceed; acknowledge it and print as usual. Only `10.00.46` / "blocked by firmware"
  means you're actually blocked.
- **No prompt on USB insert:** re-check *Allow Updates = Yes* (a failed/aborted attempt resets it
  to No); try a smaller/older FAT32 stick; some users needed a cold reset first.
- **Network push methods (9100 PJL / HP Firmware Updater utility / EWS) fail on recent
  firmware:** the manual update channel reports `UpdateLockState=on` (HP policy lock) and
  silently discards images. The USB method bypasses this. (EWS HTTPS also requires TLS 1.2.)
- **Prevent re-updates:** besides the panel settings, HP fetches firmware via
  `fw.auth.hp.com` / `prfw-fw.auth.hp.com` — blocking these in your DNS (Pi-hole/DNSBL) is a
  reliable backstop.
- **The `10.00.46` block is chip-generation based:** very old third-party chips may need
  newer stock; chips that failed 2026 firmware pass 2247A.

## Legal note

Downgrading firmware on hardware you own does not void your warranty (HP may only decline
support for bugs on old firmware). HP's Dynamic Security practices were the subject of FTC
action, the NY Attorney General's suit (settled 2023), and the *Hix v. HP* class settlement
($33.5M). The firmware image is HP's unmodified signed update, redistributed as HP's license
permits.
