# Tarish

**File sharing for de-Googled Android that works with AirDrop and Quick Share — no Google Play
Services, sandboxed or otherwise, and no Google account.**

Send a file to a Mac or an iPhone from a phone that has never spoken to Google. The Apple device
shows a real name and a normal AirDrop prompt; the phone shows a normal share sheet. Send to a
Windows PC or another Android phone and Quick Share does the same. Nothing signs in, nothing
checks in, no Google application is installed.

Everything here is verified on hardware, under SELinux **enforcing**, on a build with **zero
Google apps**. Interop is tested against **real peers** — macOS, iOS, Windows 11 and stock
Android — not only against ourselves.

**Project site: [tarish.dev](https://tarish.dev)**

---

## What works

|  | AirDrop (Apple) | Quick Share (Android / Windows) |
|---|---|---|
| discovery | AWDL + mDNS | mDNS on Wi-Fi, BLE off-network |
| send / receive | both | both |
| shared network | ✅ | ✅ **22 MB/s** |
| off-network | ✅ AWDL is its own link | ✅ Wi-Fi Direct **10.5 MB/s**, Bluetooth bootstrap |

Every number is a measurement from the project's own docs.

---

## The repositories

### 📱 [tarish-app](https://github.com/tarish-dev/tarish-app)
The Android app **and the home of all project documentation** — what Tarish is, why it exists,
which devices it works on, how to integrate it into GrapheneOS or LineageOS, the protocol notes
and the open issues. The app itself does the parts only the framework can: the share sheet and
consent, Bluetooth LE, Wi-Fi Direct groups, and handing the daemon a socket. Start here.

### ⚙️ [tarish-daemon](https://github.com/tarish-dev/tarish-daemon)
The transports, in Rust. `tarishd` holds the AWDL link up; `tarishsharingd` runs mDNS, TLS, the
AirDrop protocol and the Quick Share transports as its own unprivileged uid. `libtarish_protocol`
— UKEY2, the secure channel, the sharing state machines — is deliberately Android-free and
carries **202 tests** that run on a build host with no phone attached, including a whole share
between two peers in-process.

### 📡 [tarish-libawdl](https://github.com/tarish-dev/tarish-libawdl)
An open, clean-room implementation of **AWDL** — Apple's peer-to-peer Wi-Fi, the link AirDrop
rides on — built from over-the-air captures to replace Google's closed `libmosey`. It no longer
just parses AWDL: on a Pixel, with Google's daemon killed, it brings the radio up over netlink,
**wins Apple's master election** against real iPhones, **synchronises** to an Apple device's
schedule, and **carries IP** — with no `libmosey` in the path.

---

## The stack — every layer but one is ours

AirDrop and Quick Share are each a tower of protocols. Tarish implements the tower top to bottom
and stops only at the radio the vendor ships:

```
  share sheet & consent            app        ours
  AirDrop  (mDNS, TLS, HTTP)       daemon     ours
  Quick Share (UKEY2, upgrade)     daemon     ours
  secure channel & crypto          daemon     ours
  AWDL  (election, sync, channels) libawdl    ours   ← was Google's libmosey
  802.11 bring-up & injection      libawdl    ours
  radio driver & firmware          wonder.ko  vendor ← the one closed layer
```

---

## Licence

Apache 2.0. Tarish is an independent project and is not affiliated with or endorsed by Apple,
Google or Samsung. AirDrop is a trademark of Apple Inc.; Quick Share, Android and Pixel are
trademarks of their respective owners.
