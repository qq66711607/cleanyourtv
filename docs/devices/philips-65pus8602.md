# Philips 65PUS8602/12: as-found disabled state, four tested disables, and a verified root path

Checked on the owner's set in China (a European /12 sales model). The standard `pm disable-user` channel works normally on this firmware, and the four packages in [`examples/philips-65pus8602`](../../examples/philips-65pus8602/debloat.txt) were disabled with it and lived with. The list is short for an honest reason: this set was **not factory-fresh** when it was captured, so most of the usual candidates were already off and no tested credit is claimed for them. This note records that as-found state, and a root path that was verified on this exact model.

## Identify the exact firmware

| Field | Observed value |
|---|---|
| Retail model | Philips `65PUS8602/12`, Europe (owner's set in China) |
| Board | `TPM171E` |
| Product | `PH7M_EU_5596` |
| SoC | MediaTek MT5891 |
| Android / SDK | Android TV 8.0.0 / 26 |
| Build | `OC` incremental `4718` |
| Android security patch | 2021-01-05 |
| Fingerprint | `Philips/PH7M_EU_5596/PH7M_EU_5596:8.0.0/OC/4718:user/release-keys` |

The fingerprint family matches [`philips-55pus7503`](../../examples/philips-55pus7503/debloat.txt) and [`philips-65pus8102`](../../examples/philips-65pus8102/debloat.txt): TPM171E, `PH7M_EU_5596`, Android TV 8.0, security patch 2021-01-05. This is a different sales model and a differently-aged set, so those lists were used as a candidate source, not applied wholesale.

## The as-found state (2026-10-18, before any change by us)

The set had been the owner's daily TV for years and was not factory-reset before capture. `pm list packages` showed 152 packages installed; `pm list packages -d` already showed **44 packages disabled** in that untouched state:

```
com.amazon.amazonvideo.livingroom    com.google.android.tvtutorials       org.droidtv.nettvapp5326
com.android.dreams.basic             com.google.android.videos            org.droidtv.nettvapp5512
com.android.printspooler             com.google.android.youtube.tv        org.droidtv.nettvapps
com.android.vending                  com.mediatek.dmagent                 org.droidtv.nettvbrowser
com.gameloft.android.HEP.GloftANHP   com.netflix.ninja                    org.droidtv.nettvrecommender
com.google.android.backuptransport   com.opera.sdk.example                org.droidtv.sticker
com.google.android.feedback          org.droidtv.amazonalexa              org.droidtv.teletext
com.google.android.katniss           org.droidtv.candeebug                org.droidtv.twittershare
com.google.android.leanbacklauncher.recommendations                       org.droidtv.usagelogger
com.google.android.music             org.droidtv.ci                       tv.inscape.tvclient.inscapeacr
com.google.android.play.games        org.droidtv.demome
com.google.android.syncadapters.calendar   org.droidtv.dropboxprovider
com.google.android.syncadapters.contacts   org.droidtv.leanbackcustomizer
com.google.android.tvrecommendations       org.droidtv.nettv.market
                                     org.droidtv.nettv_launcherapp
                                     org.droidtv.nettvapp1814
                                     org.droidtv.nettvapp1984
                                     org.droidtv.nettvapp20004410
                                     org.droidtv.nettvapp4655
                                     org.droidtv.nettvapp4771
                                     org.droidtv.nettvapp5225
```

We cannot say which of these the firmware shipped with disabled, a past update changed, or an earlier setup step turned off — only that all 44 predate our first change. Twenty-two of them overlap the candidate set we had picked from the sibling lists (Google recommendation/feedback rows, Philips telemetry, the NetTV store and web apps, the Opera sample, Inscape ACR). Owners comparing sets should expect their as-found state to differ: on the [`philips-49pus8303`](philips-49pus8303.md) set with the same build, only `com.google.android.tvlauncher` was disabled.

Two as-found entries deserve a warning rather than silent copying:

- `org.droidtv.nettvbrowser` — the 65PUS8102 list keeps this enabled on purpose because it is the HbbTV handler. On this set it was already off; this household watches IPTV only and never uses broadcast or HbbTV, so the red button is **not validated** here.
- `com.android.vending`, `com.google.android.youtube.tv`, `com.netflix.ninja` were also already off; the owner uses the Play Store, YouTube (via other clients) and Netflix through other means. Nothing in this note tests re-enabling them.

## What we actually disabled

On 2026-10-18, after installing a replacement launcher and setting it as Home, we disabled the four packages in the example list: `com.google.android.tvlauncher` (stock launcher), `org.droidtv.nettvregistration`, `com.mediatek.TimeMeasurementAgent` and `com.example.tvcontentprovider`. The set has been in daily use since — replacement Home, IPTV live TV, streaming apps, Bluetooth audio — with no breakage attributed to these four. The capture, the disable output and a restore script covering exactly these four were kept from that session.

## Root: verified on this model (not needed for the list)

The debloat above was done over plain ADB **before** any rooting; root is not required for it. Separately, this platform is rootable, and it was carried out and verified on this set on 2026-10-19/20:

- The method is the public **[yath/tpm171e](https://github.com/yath/tpm171e)** project — the root path is that author's work; the contribution here is only the verification on this sales model (the project's reference set is a 65OLED873 on the same TPM171E platform).
- In outline: the main driver's debug interface `/dev/cli` is writable by the ordinary shell user; the project's Loader uses it to gain root, and its `install-magisk` then patches the boot partition with Magisk (the stock boot image is first backed up to the unused recoveryB partition).
- Verified state afterwards: Magisk installed, SELinux enforcing, the ADB shell granted root. The set has run rooted in daily use since.
- A factory reset does **not** remove this root (it lives in the boot partition, which a reset does not touch), but everything Magisk keeps under `/data` — modules, grant records, the manager app — is wiped and must be set up again. Removing root means flashing the backed-up stock boot image back.
- What root was used for on this set, as an example: an RRO overlay module that flips the stock `profile_supported_a2dp=false` configuration, after which the set connects to Bluetooth speakers as an A2DP source.

## Validation limits

- The model, firmware and package states above were read from the set; the as-found capture is dated 2026-10-18 and was taken before our first change, but the set was not factory-fresh, so the 44 disabled packages are an observation, not a factory specification.
- Only the four packages in the example list are claimed as disabled-and-lived-with by us (from 2026-10-18 onward).
- Broadcast, HbbTV and the tuner path are unused on this set and are not validated; Live TV here means IPTV.
- The root outline is a summary, not a guide: follow the tpm171e project's own documentation, and expect the boot-partition write to be the risky step.
