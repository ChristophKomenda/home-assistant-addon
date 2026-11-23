<div align="center">

# Pironman 5 Max — IR Receiver for Home Assistant

**Community-maintained Home Assistant OS add-on · Native infrared reception · HAIR support**

Use the **built-in IR receiver on GPIO13** of the SunFounder Pironman 5 Max in Home Assistant, without extra IR receiver hardware. Infrared signals are captured locally via LIRC and exposed to HAIR through either the existing Pironman 5 HACS integration (recommended) or the legacy standalone companion.

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
| **Add-on version** | 0.2.0 (see `pironman-ir-receiver/config.yaml`) |
| **Device** | `/dev/lirc0` (must exist before starting the add-on) |
| **Companion integration** | **Pironman 5 v1.5.0+ via HACS** (recommended); manual `sunfounder_ir` only for standalone mode |
| **Optional experience** | [HAIR Sniffer](https://github.com/Anirudh-G/HAIR) for learning IR signals |
| **Data flow** | Fully local; no MQTT, cloud or public TCP port required |

**Receive-only:** This add-on cannot transmit IR signals. Successfully capturing codes in HAIR does not mean the Pironman can send those codes back to other devices.

## Why this add-on?

- **Use hardware you already own:** Receive IR signals through the Pironman 5 Max's integrated receiver.
- **Works with Home Assistant's infrared platform:** In `pironman5` mode the HACS integration creates the IR receiver entity; standalone mode can still use the separate `sunfounder_ir` component.
- **Local by design:** LIRC frames are delivered to Home Assistant using its Supervisor API; no cloud account or MQTT broker.
- **Focused permissions:** The add-on maps `/dev/lirc0` rather than requesting blanket privileged hardware access.
- **Community-tested:** End-to-end IR capture was verified locally with Home Assistant Container and independently on HAOS (see below).

## Quick start (Home Assistant OS)

For **Home Assistant OS**, the recommended setup uses this community IR add-on
with the [Pironman 5 HACS integration](https://github.com/ChristophKomenda/pironman5-homeassistant-container)
(v1.5.0 or newer). **There is no need to copy the separate `sunfounder_ir`
custom integration.** The original standalone option remains supported.

1. Make sure the Pironman 5 Max IR receiver is available as `/dev/lirc0`
   (GPIO13 with the appropriate `gpio-ir` overlay).
2. Install the Pironman 5 integration through HACS, configure its reachable
   device REST API, and enable the experimental IR receiver option.
3. Install or update **Pironman IR Receiver v0.2.0** from this community
   add-on repository. Configure:

   ```yaml
   receiver_id: pironman5-max
   event_mode: pironman5
   debug: false
   ```

4. Set the Pironman 5 integration's **IR bridge device ID** to the same value,
   here `pironman5-max`. Disable the former `sunfounder_ir` integration if it
   was previously installed, to avoid duplicate receiver entities.
5. **Reload the Pironman 5 integration** after changing the add-on or IR mode.
   This step was necessary in independent HAOS testing when HAIR initially
   showed no working receiver.
6. In **HAIR > Sniffer**, select the receiver provided by **Pironman 5** and
   try capturing a remote-control signal.

**Independent HAOS validation (23 November 2026):**
[@eugenioparise](https://github.com/eugenioparise) reported that updating the
add-on, selecting `event_mode: pironman5`, disabling the old `sunfounder_ir`
integration, reloading Pironman 5, and matching both receiver IDs resulted in
successful HAIR signal capture. HAIR showed only the Pironman 5 integration's
receiver. The tester also reported successful HAIR send functionality; the
Pironman GPIO IR hardware in this add-on remains **receive-only** (sending
codes requires a separate compatible transmitter). See the
[community discussion](https://github.com/ChristophKomenda/pironman5-homeassistant-container/discussions/16).

This confirms one real HAOS installation, not universal compatibility across
all Home Assistant versions or Pironman variants.

## Alternative: standalone mode

Set `event_mode: standalone` and install the companion `sunfounder_ir`
custom integration manually, as documented in the
[standalone instructions](pironman-ir-receiver/README.md#legacy-standalone-installation-v011).
This is optional and **not needed** for the recommended HACS setup.

## Tested environments

| Environment | Result | Evidence |
| --- | --- | --- |
| Raspberry Pi 5 / Pironman 5 Max / Debian / Home Assistant Container | IR reception and native entity shown in HAIR | [Technical details](pironman-ir-receiver/README.md#live-end-to-end-validation-on-home-assistant-container-21-november-2026) |
| Pironman 5 Max / HAOS 18.5 / Home Assistant Core 2026.11.2 | Standalone mode independently tested with real IR capture | [Community test](https://github.com/ChristophKomenda/pironman5-homeassistant-container/discussions/16) |
| Pironman 5 Max / HAOS (new HACS-linked mode) | Independent tester reports successful HAIR capture after reloading Pironman 5 | [Community report](https://github.com/ChristophKomenda/pironman5-homeassistant-container/discussions/16) |

The original standalone HAOS test was reported by [@eugenioparise](https://github.com/eugenioparise) on **November 21, 2026**. The new HACS-linked mode was independently reported working on **November 23, 2026**, including HAIR after an integration reload. Broader hardware compatibility, LIRC device paths other than `/dev/lirc0`, and long-term reliability remain unverified.

## Related projects and upstream status

- **This repository:** Independently maintained **HAOS IR add-on**, with optional standalone `sunfounder_ir` companion.
- **[Pironman 5 HACS integration](https://github.com/ChristophKomenda/pironman5-homeassistant-container):** Recommended companion for `event_mode: pironman5` on HAOS; also supports a separate host IR bridge on Home Assistant Container.
- **[SunFounder PR #26](https://github.com/sunfounder/home-assistant-addon/pull/26):** Standalone add-on submission; SunFounder declined upstream maintenance.
- **[SunFounder PR #27](https://github.com/sunfounder/home-assistant-addon/pull/27):** Proposed community add-on listing in SunFounder's README. Managed independently here; not an official SunFounder-maintained add-on.

This fork retains other SunFounder add-on directories for compatibility with its upstream structure. For **official SunFounder-maintained add-ons**, prefer [SunFounder's repository](https://github.com/sunfounder/home-assistant-addon). This community repository primarily focuses on the experimental Pironman IR Receiver.

## Support and contributions

- **Installation or IR receiver issue:** [Open an issue in this repository](https://github.com/ChristophKomenda/home-assistant-addon/issues).
- **Community testing and feedback:** [Discussion and independent HAOS test](https://github.com/ChristophKomenda/pironman5-homeassistant-container/discussions/16).
- **Debugging:** [IR troubleshooting and logs](pironman-ir-receiver/README.md#diagnostics-optional-debug-logging).

When reporting a problem, include your Pironman model, HAOS and Home Assistant Core versions, whether `/dev/lirc0` exists, add-on version, event mode, integration reload status and **redacted** log excerpts. Never share access tokens or private configuration data.

Contributions and additional independent hardware test reports are welcome. For IR-specific changes, tests can be run with:

```bash
python3 -m unittest discover -s pironman-ir-receiver/tests -v
```

---

*Community-maintained by [Christoph Komenda](https://github.com/ChristophKomenda). Not affiliated with or officially supported by SunFounder.*
