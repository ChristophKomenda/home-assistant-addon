<div align="center">

# Pironman 5 Max — IR Receiver for Home Assistant OS

**Use the built-in GPIO13 infrared receiver in Home Assistant and HAIR.**

Community-maintained · Local IR reception · No external IR receiver required · **Receive-only**

[![SunFounder community listing](https://img.shields.io/badge/SunFounder-community%20listed-blue)](https://github.com/sunfounder/home-assistant-addon#community-add-ons)
[![HACS integration](https://img.shields.io/badge/HACS-default%20integration-41BDF5)](https://github.com/ChristophKomenda/pironman5-homeassistant-container)


[**Install add-on repository**](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2FChristophKomenda%2Fhome-assistant-addon) · [**Step-by-step setup**](#quick-start-recommended-setup) · [**Troubleshooting**](pironman-ir-receiver/README.md#diagnostics-optional-debug-logging) · [**Technical details**](pironman-ir-receiver/README.md)

[![Add repository to Home Assistant](https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg)](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2FChristophKomenda%2Fhome-assistant-addon)

</div>

> [!IMPORTANT]
> **Independent community add-on — not maintained or officially supported by SunFounder.** This repository is a fork of [SunFounder's add-on repository](https://github.com/sunfounder/home-assistant-addon). It primarily maintains the **Pironman IR Receiver** add-on; for SunFounder's original add-ons, use their official repository. See [upstream status](#upstream-and-project-status).

## At a glance

| | |
| --- | --- |
| **Hardware** | Pironman 5 Max with built-in IR receiver |
| **Platform** | Home Assistant OS (ARM64) |
| **Recommended setup** | IR add-on v0.2.0+ and Pironman 5 HACS integration v1.5.0+ |
| **Validation** | Independent HAOS and HAIR community testing |
| **Recognition** | Listed in SunFounder's official repository via [merged PR #27](https://github.com/sunfounder/home-assistant-addon/pull/27) on November 24, 2026 |
| **Privacy** | Local processing without cloud dependencies |
| **IR capability** | Receive-only |

**Choose your path:**

- [New HAOS installation](#quick-start-recommended-setup)
- [Existing standalone users](#which-mode-should-i-use)
- [Home Assistant Container users](https://github.com/ChristophKomenda/pironman5-homeassistant-container/blob/main/docs/native-ir-hair.md)
- [Compatibility and requirements](#before-you-start)

## What can I do with it?

- **Receive IR remote signals** through the Pironman 5 Max's built-in GPIO13 IR receiver.
- **Learn signals in HAIR** through a native Home Assistant infrared receiver entity.
- **Keep everything local:** no cloud account, MQTT broker or public TCP port is needed for IR event delivery.
- **Choose an integration mode:** the existing [Pironman 5 HACS integration](https://github.com/ChristophKomenda/pironman5-homeassistant-container) is recommended; the earlier standalone companion is still available.

**Important:** This add-on **cannot transmit infrared signals**. HAIR can capture/learn codes with this receiver, but sending them requires separate compatible transmitter hardware.

## Before you start

| Requirement | Why it matters |
| --- | --- |
| **Raspberry Pi 5 + SunFounder Pironman 5 Max** | The built-in GPIO13 IR receiver was tested on this model. Other models are not yet verified. |
| **Home Assistant OS (aarch64)** | This repository provides an **HAOS add-on**, not a host-service installer for Container. |
| **Home Assistant Core 2026.6.0+** | Required for the native infrared receiver platform. |
| **Working `/dev/lirc0` on the HAOS host** | The add-on reads this device; GPIO13 must be configured with the appropriate `gpio-ir` overlay. |
| **Pironman 5 HACS integration v1.5.0+** | Recommended mode uses its native IR entity and a reachable local Pironman REST API (usually port `34001`). |
| **HAIR (optional)** | Install it if you want to sniff and learn remote-control signals. |

> [!WARNING]
> **GPIO setup is a prerequisite, not something HACS or this add-on installs for you.** If `/dev/lirc0` does not exist, the add-on may fail to start. The appropriate `dtoverlay=gpio-ir,gpio_pin=13` configuration must be applied for *your* HAOS environment; do not modify boot partitions blindly. See the [requirements and diagnostics](pironman-ir-receiver/README.md#requirements).

## Quick start: recommended setup

**New to this project? Choose `event_mode: pironman5`.** This connects the IR add-on to the normal Pironman 5 HACS integration; you do **not** need to copy `sunfounder_ir` manually.

1. **Check the receiver:** Confirm that the HAOS host exposes `/dev/lirc0` after configuring GPIO13 / LIRC.
2. **Install the HACS integration:** In HACS search for **Pironman 5** (v1.5.0+). Add your Pironman under **Settings → Devices & Services**, using the reachable REST API address and port (normally `34001`).
3. **Enable IR in the integration:** Open **Pironman 5 → Configure**, enable **experimental native infrared receiver**, and set **IR bridge device ID** to `pironman5-max` (or another unique ID).
4. **Add this community repository to HAOS:** Open the [add-on repository link](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2FChristophKomenda%2Fhome-assistant-addon) or open **Settings → Add-ons / Apps → Add-on Store → ⋮ → Repositories**, then enter:

   ```text
   https://github.com/ChristophKomenda/home-assistant-addon
   ```

5. **Install Pironman IR Receiver v0.2.0+** from that repository. Under **Configuration** select these options:

   ```yaml
   receiver_id: pironman5-max
   event_mode: pironman5
   debug: false
   ```

6. **Start the add-on.** Its `receiver_id` must match the HACS integration's **IR bridge device ID** exactly. If upgrading from `sunfounder_ir`, disable the older custom integration to avoid duplicate receivers.
7. **Reload Pironman 5** in **Settings → Devices & Services**. This was needed during the independent HAOS test after switching modes.
8. **Verify:** Open **HAIR → Sniffer**, select the receiver belonging to **Pironman 5**, then press a button on a physical IR remote. If nothing is captured, see [troubleshooting](pironman-ir-receiver/README.md#diagnostics-optional-debug-logging).

> [!TIP]
> Already using the older **standalone** installation? It remains available. You do not have to migrate unless you want to use the recommended HACS-linked mode. See [legacy standalone instructions](pironman-ir-receiver/README.md#legacy-standalone-installation).

## How the pieces fit together

```text
IR remote → Pironman 5 Max GPIO13 → LIRC /dev/lirc0
                                  → Pironman IR Receiver HAOS add-on
                                  → Home Assistant event bus
                                  → Pironman 5 HACS integration
                                  → Native IR receiver → HAIR Sniffer
```

**Two separate installations, one receiver:** HACS installs the integration; the HAOS add-on reads the hardware and forwards frames. You need both for the recommended IR path. Ordinary Pironman monitoring, fan/RGB/OLED controls **do not require** this IR add-on.

## Which mode should I use?

| Mode | Best for | Additional integration |
| --- | --- | --- |
| **`pironman5` (recommended)** | New HAOS installations; native IR in the existing Pironman 5 integration | Pironman 5 v1.5.0+ via HACS; **no manual `sunfounder_ir` copy** |
| **`standalone` (legacy, default)** | Existing users of the previous IR add-on setup | Separate manually installed `sunfounder_ir` component |

**Why is `standalone` still the default?** It preserves existing user configurations. For new installations select **`pironman5` explicitly** under add-on options.

## Tested environments

| Setup | Reported result | Evidence |
| --- | --- | --- |
| Raspberry Pi 5 / Pironman 5 Max / Debian / Home Assistant Container | Local IR-to-HAIR reception demonstrated with a host-based implementation | [Technical details](pironman-ir-receiver/README.md#live-end-to-end-validation-on-home-assistant-container-21-november-2026) |
| Pironman 5 Max / HAOS 18.5 / Core 2026.11.2, standalone mode | Independent IR capture and HAIR testing | [Community report](https://github.com/ChristophKomenda/pironman5-homeassistant-container/discussions/16) |
| Pironman 5 Max / HAOS, `pironman5` mode | Independent IR capture in HAIR following integration reload (23 November 2026) | [Community report](https://github.com/ChristophKomenda/pironman5-homeassistant-container/discussions/16) |

Thanks to [@eugenioparise](https://github.com/eugenioparise) for independently testing both HAOS modes. Results are specific to the tested environments and do not establish compatibility with every HAOS release or Pironman model.

## Help and troubleshooting

If the add-on does not start, **check `/dev/lirc0` first**. If it starts but HAIR sees no signals, check the exact receiver ID, `event_mode`, Pironman integration IR option, and whether you reloaded the integration. For more detailed log diagnostics, use the [full troubleshooting section](pironman-ir-receiver/README.md#diagnostics-optional-debug-logging).

- [Detailed setup and historical standalone instructions](pironman-ir-receiver/README.md)
- [Report a problem in the community discussion](https://github.com/ChristophKomenda/pironman5-homeassistant-container/discussions/16)
- [Discuss hardware testing](https://github.com/ChristophKomenda/pironman5-homeassistant-container/discussions/16)
- [Release v0.2.0](https://github.com/ChristophKomenda/home-assistant-addon/releases/tag/pironman-ir-v0.2.0)
- [Run automated add-on tests](pironman-ir-receiver/README.md#development-tests)

When reporting problems, include your device model, HAOS/Core versions, add-on version, event mode, `/dev/lirc0` status and **redacted** logs. Never publish tokens or private configuration details.

## Upstream and project status

### Listed in SunFounder's official repository

On November 24, 2026, SunFounder merged [PR #27](https://github.com/sunfounder/home-assistant-addon/pull/27), listing this independently maintained IR receiver in the [Community add-ons section of its official repository](https://github.com/sunfounder/home-assistant-addon#community-add-ons).

The listing followed independent HAOS testing, technical collaboration and installation improvements based on maintainer feedback.

| Milestone | Outcome |
| --- | --- |
| Community demand | Identified the need for native IR reception on HAOS |
| Engineering | Developed a standalone IR receiver add-on |
| Hardware validation | Independent HAOS and HAIR testing |
| Maintainer feedback | Identified installation and maintenance concerns |
| Installation UX | Removed manual companion integration installation in v0.2.0 |
| Official documentation | SunFounder merged PR #27 |

The original [upstream PR #26](https://github.com/sunfounder/home-assistant-addon/pull/26) was not merged.

The project remains independently maintained. Its listing does not constitute official SunFounder support or endorsement.

The add-on remains experimental, independently maintained and receive-only. Contributions and reports from additional physical HAOS installations are welcome.

## Frequently asked questions

<details>
<summary><strong>Do I need to manually install a separate custom integration?</strong></summary>

No. The recommended `pironman5` mode connects directly to the existing Pironman 5 HACS integration.

Only the legacy standalone mode requires the separate `sunfounder_ir` component.

</details>

<details>
<summary><strong>Can I transmit infrared commands to my TV?</strong></summary>

No. The built-in receiver and this add-on are receive-only. Sending infrared commands requires separate compatible transmitter hardware.

</details>

<details>
<summary><strong>Do I need this add-on for normal Pironman monitoring?</strong></summary>

No. Hardware monitoring, fan control, RGB lighting and OLED controls do not require the IR receiver add-on.

</details>

<details>
<summary><strong>Does this work with Home Assistant Container?</strong></summary>

This add-on is intended for Home Assistant OS.

Container users should follow the [native IR and HAIR documentation](https://github.com/ChristophKomenda/pironman5-homeassistant-container/blob/main/docs/native-ir-hair.md) and use the documented Linux host bridge.

</details>

<details>
<summary><strong>Is this officially supported by SunFounder?</strong></summary>

No. SunFounder lists this project in its official repository, but maintenance and support remain independent.

</details>

<details>
<summary><strong>How can I contribute?</strong></summary>

Additional hardware validation, reproducible bug reports and documentation improvements are welcome.

Share your experience in the [community discussion](https://github.com/ChristophKomenda/pironman5-homeassistant-container/discussions/16).

Never include access tokens or sensitive configuration data in public reports.

</details>

## License and attribution

Original contributions in [`pironman-ir-receiver/`](pironman-ir-receiver/) are MIT-licensed **to the extent of the maintainer's rights**. See the add-on's [LICENSE](pironman-ir-receiver/LICENSE) and [copyright scope](pironman-ir-receiver/COPYRIGHT.md). This does not relicense SunFounder's original code, other files in the parent fork or third-party materials.

---

*Community-maintained by [Christoph Komenda](https://github.com/ChristophKomenda). Not affiliated with or officially supported by SunFounder.*

<!-- github-growth-related-projects -->
## Related projects

- [**Home Assistant Community Apps**](https://github.com/ChristophKomenda/home-assistant-apps) — Central app catalog; runtime validation pending.
