<div align="center">

# Pironman 5 Max — IR Receiver for Home Assistant

**Community-maintained Home Assistant OS add-on · Native infrared reception · HAIR support**

Use the **built-in IR receiver on GPIO13** of the SunFounder Pironman 5 Max in Home Assistant, without extra IR receiver hardware. Infrared signals are captured locally via LIRC, forwarded as Home Assistant events and exposed by a separate companion integration as a native `InfraredReceiverEntity`.

[**Quick start**](#quick-start-home-assistant-os) · [**Full installation guide**](pironman-ir-receiver/README.md) · [**Testing**](#tested-environments) · [**Troubleshooting**](pironman-ir-receiver/README.md#diagnostics-optional-debug-logging) · [**Support**](#support-and-contributions)

</div>

> [!IMPORTANT]
> **Independent community project.** This repository is maintained by [@ChristophKomenda](https://github.com/ChristophKomenda), not by SunFounder. SunFounder [PR #26](https://github.com/sunfounder/home-assistant-addon/pull/26) was not accepted for upstream maintenance; [PR #27](https://github.com/sunfounder/home-assistant-addon/pull/27) proposes linking this independently maintained add-on from the official README. PR #27 is a community listing, **not** an official SunFounder release or endorsement.

## At a glance

| | |
| --- | --- |
| **Hardware** | SunFounder Pironman 5 Max with built-in IR receiver on GPIO13 |
| **Deployment** | Home Assistant OS (HAOS) on Raspberry Pi 5, aarch64 |
| **Minimum Home Assistant Core** | 2026.6.0 (native infrared receiver platform required) |
| **Add-on version** | 0.1.1 (experimental; check `pironman-ir-receiver/config.yaml`) |
| **Device** | `/dev/lirc0` (must exist before starting the add-on) |
| **Companion integration** | `sunfounder_ir`, installed **manually** into `/config/custom_components/` |
| **Optional experience** | [HAIR Sniffer](https://github.com/Anirudh-G/HAIR) for learning IR signals |
| **Data flow** | Fully local; no MQTT, cloud or public TCP port required |

**Receive-only:** This add-on cannot transmit IR signals. Successfully capturing codes in HAIR does not mean the Pironman can send those codes back to other devices.

## Why this add-on?

- **Use hardware you already own:** Receive IR signals through the Pironman 5 Max's integrated receiver.
- **Works with Home Assistant's infrared platform:** The separate custom integration creates an `InfraredReceiverEntity`, usable in HAIR Sniffer.
- **Local by design:** LIRC frames are delivered to Home Assistant using its Supervisor API; no cloud account or MQTT broker.
- **Focused permissions:** The add-on maps `/dev/lirc0` rather than requesting blanket privileged hardware access.
- **Community-tested:** End-to-end IR capture was verified locally with Home Assistant Container and independently on HAOS (see below).

## Quick start (Home Assistant OS)

> [!CAUTION]
> This procedure is for **Pironman 5 Max on HAOS**, with `/dev/lirc0` already available. Kernel overlays and boot-file locations differ by installation. Do not edit or reflash a boot partition without first verifying the correct procedure and making a backup. The add-on will not start without the expected LIRC device.

1. In Home Assistant, open **Settings → Add-ons / Apps → Add-on Store → Repositories** (menu wording varies by HA version).
2. Add the community repository URL:

   ```text
   https://github.com/ChristophKomenda/home-assistant-addon
   ```

3. Refresh the store, install **Pironman IR Receiver**, and verify that the installed version matches the [current add-on configuration](pironman-ir-receiver/config.yaml).
4. Set the add-on options (for example `receiver_id: pironman5-max`; leave `debug: false` unless troubleshooting), then start it and check logs.
5. **Install the companion integration separately:** Copy [`pironman-ir-receiver/ha_custom_component/sunfounder_ir/`](pironman-ir-receiver/ha_custom_component/sunfounder_ir/) to Home Assistant's `/config/custom_components/sunfounder_ir/` and restart Home Assistant. The add-on **does not** install the integration automatically.
6. Go to **Settings → Devices & Services → Add Integration → SunFounder IR Receiver** and use the **same receiver ID** as in the add-on.
7. If you use HAIR, open **HAIR → Sniffer**, select **SunFounder IR Receiver**, and press a button on an IR remote.

**Full instructions and troubleshooting:** [Pironman IR Receiver README](pironman-ir-receiver/README.md). If the add-on refuses to start, verify `/dev/lirc0` exists and that no conflicting service is consuming the receiver.

> [!NOTE]
> If you also use the [Pironman 5 HACS integration](https://github.com/ChristophKomenda/pironman5-homeassistant-container), disable its optional experimental IR receiver for that hardware before using this standalone add-on. Both implementations should not compete for the same LIRC device. This standalone add-on does not otherwise depend on the HACS integration.

## Architecture

```text
IR remote
   │
   ▼
Pironman 5 Max GPIO13 → /dev/lirc0 (LIRC)
   │
   ▼
Pironman IR Receiver (HAOS add-on)
   │  local Supervisor API: sunfounder_ir_received
   ▼
Home Assistant event bus
   │
   ▼
sunfounder_ir (separate custom integration)
   │
   ▼
InfraredReceiverEntity → HAIR Sniffer (optional)
```

## Tested environments

| Environment | Result | Evidence |
| --- | --- | --- |
| Raspberry Pi 5 / Pironman 5 Max / Debian / Home Assistant Container | IR reception and native entity shown in HAIR | [Technical details](pironman-ir-receiver/README.md#live-end-to-end-validation-on-home-assistant-container-21-november-2026) |
| Pironman 5 Max / HAOS 18.5 / Home Assistant Core 2026.11.2 | Independently reported end-to-end success, including real remote capture | [Community test](https://github.com/ChristophKomenda/pironman5-homeassistant-container/discussions/16) |

The independent HAOS test was reported by [@eugenioparise](https://github.com/eugenioparise) on **November 21, 2026**. Broader hardware compatibility, LIRC device paths other than `/dev/lirc0`, and long-term reliability remain unverified.

## Related projects and upstream status

- **This repository:** Independently maintained experimental **HAOS add-on and `sunfounder_ir` companion integration**.
- **[Pironman 5 HACS integration](https://github.com/ChristophKomenda/pironman5-homeassistant-container):** Separate hardware monitoring/control integration with an optional *different* IR host bridge for Home Assistant Container. Not required for this add-on.
- **[SunFounder PR #26](https://github.com/sunfounder/home-assistant-addon/pull/26):** Standalone add-on submission; SunFounder declined upstream maintenance.
- **[SunFounder PR #27](https://github.com/sunfounder/home-assistant-addon/pull/27):** Proposed community add-on listing in SunFounder's README. Managed independently here; not an official SunFounder-maintained add-on.

This fork retains other SunFounder add-on directories for compatibility with its upstream structure. For **official SunFounder-maintained add-ons**, prefer [SunFounder's repository](https://github.com/sunfounder/home-assistant-addon). This community repository primarily focuses on the experimental Pironman IR Receiver.

## Support and contributions

- **Installation or IR receiver issue:** [Open an issue in this repository](https://github.com/ChristophKomenda/home-assistant-addon/issues).
- **Community testing and feedback:** [Discussion and independent HAOS test](https://github.com/ChristophKomenda/pironman5-homeassistant-container/discussions/16).
- **Debugging:** [IR troubleshooting and logs](pironman-ir-receiver/README.md#diagnostics-optional-debug-logging).

When reporting a problem, include your Pironman model, HAOS and Home Assistant Core versions, whether `/dev/lirc0` exists, add-on version, companion integration status and **redacted** log excerpts. Never share access tokens or private configuration data.

Contributions and additional independent hardware test reports are welcome. For IR-specific changes, tests can be run with:

```bash
python3 -m unittest discover -s pironman-ir-receiver/tests -v
```

---

*Community-maintained by [Christoph Komenda](https://github.com/ChristophKomenda). Not affiliated with or officially supported by SunFounder.*
