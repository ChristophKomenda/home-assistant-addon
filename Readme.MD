<div align="center">

# Pironman 5 Max — IR Receiver for Home Assistant OS

**Use the built-in GPIO13 infrared receiver in Home Assistant and HAIR.**

Community-maintained · Local IR reception · No external IR receiver required · **Receive-only**

[**Install add-on repository**](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2FChristophKomenda%2Fhome-assistant-addon) · [**Step-by-step setup**](#quick-start-recommended-setup) · [**Troubleshooting**](pironman-ir-receiver/README.md#diagnostics-optional-debug-logging) · [**Technical details**](pironman-ir-receiver/README.md)

</div>

> [!IMPORTANT]
> **Independent community add-on — not maintained or officially supported by SunFounder.** This repository is a fork of [SunFounder's add-on repository](https://github.com/sunfounder/home-assistant-addon). It primarily maintains the **Pironman IR Receiver** add-on; for SunFounder's original add-ons, use their official repository. See [upstream status](#upstream-and-project-status).

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
- [Report an issue](https://github.com/ChristophKomenda/home-assistant-addon/issues)
- [Discuss hardware testing](https://github.com/ChristophKomenda/pironman5-homeassistant-container/discussions/16)
- [Release v0.2.0](https://github.com/ChristophKomenda/home-assistant-addon/releases/tag/pironman-ir-v0.2.0)
- [Run automated add-on tests](pironman-ir-receiver/README.md#development-tests)

When reporting problems, include your device model, HAOS/Core versions, add-on version, event mode, `/dev/lirc0` status and **redacted** logs. Never publish tokens or private configuration details.

## Upstream and project status

SunFounder declined maintaining this add-on in [upstream PR #26](https://github.com/sunfounder/home-assistant-addon/pull/26). [PR #27](https://github.com/sunfounder/home-assistant-addon/pull/27) proposed listing it among community-maintained projects. This is **not** an official SunFounder add-on or endorsement.

The add-on remains experimental, independently maintained and receive-only. Contributions and reports from additional physical HAOS installations are welcome.

## License and attribution

Original contributions in [`pironman-ir-receiver/`](pironman-ir-receiver/) are MIT-licensed **to the extent of the maintainer's rights**. See the add-on's [LICENSE](pironman-ir-receiver/LICENSE) and [copyright scope](pironman-ir-receiver/COPYRIGHT.md). This does not relicense SunFounder's original code, other files in the parent fork or third-party materials.

---

*Community-maintained by [Christoph Komenda](https://github.com/ChristophKomenda). Not affiliated with or officially supported by SunFounder.*
