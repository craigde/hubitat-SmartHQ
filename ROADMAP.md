# Roadmap

Tracking work that's been deliberately deferred. Issues and ideas that should be addressed in future versions live here so they don't get lost.

## Phase 2 — Component child devices

The Phase 1 capability cleanup (1.0.3) added the most useful Hubitat capabilities to each appliance driver. Phase 2 takes the next step: split appliances with multiple distinct sub-elements into proper Hubitat **component child devices** so each sub-element exposes its own native capability tile.

### Refrigerator: child ContactSensor per door

The refrigerator currently has four door attributes (`leftDoor`, `rightDoor`, `freezer`, `drawer`) on a single device. None of them drives a `ContactSensor` capability because there's no clean way to map four contacts to a single `contact` attribute. As a result, "notify me when the fridge door is open >5 minutes" automations don't work natively.

Plan:
- Create a `SmartHQ Refrigerator Door` driver (component, ContactSensor capability)
- On install/parse, the parent refrigerator creates 4 component children (one per door) and updates each child's `contact` attribute when `parseDoorStatus` fires
- The four attributes on the parent can stay (backward compat) or be removed

### Hood: child Light

The hood has `lightLevel` (off/dim/high) and `setLightLevel` but no real light control surface. Users commonly want hood light automations ("when stove is on, hood light on").

Plan:
- Create a `SmartHQ Hood Light` driver (component, Switch + SwitchLevel capability)
- Map `setLevel(0)` → `setLightLevel("off")`, `setLevel(50)` → `setLightLevel("dim")`, `setLevel(100)` → `setLightLevel("high")`
- Map `on()` to whichever of dim/high was last set (default `high`)
- Parent hood device pushes `lightLevel` updates to the child

### Ice Maker: child Light

Same pattern as hood. Ice maker has `lightLevel` (off/on/dim) and `setLightLevel`. Add a `SmartHQ Ice Maker Light` component child for direct light control.

### Oven: child Switch per oven (upper / lower)

The oven currently rolls upper and lower cook state into a single `switch` attribute. For ovens with separately-controllable upper and lower cavities, automations like "if upper oven is on" can't target them individually.

Plan:
- Create `SmartHQ Oven Cavity` component children (Switch + custom cookMode attribute)
- Parent oven creates one or two children depending on `OVEN_CONFIGURATION` ERD
- Per-cavity attributes on the parent remain for backward compat

### Cooktop: child Switch

Similar to oven cavities — `cooktopStatus` could be a child Switch component for cleaner automation targeting.

---

## Other follow-ups

### Info-level logging preference — roll out to other drivers?

Alan_F's dehumidifier driver introduces an `infoLogEnable` preference that logs each attribute change at `log.info` level (useful for external log aggregators). It's currently scoped to just the dehumidifier; consider whether to add the same preference + `logInfo(msg)` helper to the other appliance drivers for consistency.



### Release checklist: regenerate the library bundle

Whenever `libraries/smarthqHelpers` changes, the `SmartHQHelpersLibrary.zip` bundle at the repo root **must** be regenerated and recommitted. HPM installs bundles first, so a stale bundle keeps an out-of-date library on the user's hub regardless of what's in `main`. Symptom of forgetting: users see errors related to old library code paths (e.g. the `getDataValue("userId")` fallback that used to live in `getUserId()`).

Steps to regenerate are in the README under "Regenerating the Bundle".

### Token / WSS credential debug logging (security)

Currently the app logs full OAuth token responses and WSS endpoint URLs (with embedded access tokens) when debug logging is on. This was kept enabled for v1 troubleshooting per the original review. **Before any wider release**, change these `logDebug` calls in `apps/smartHQ_app` to log success/failure messages only:

```groovy
def refresh = get_oauth2_token("refresh")
logDebug("token refresh succeeded")  // NOT logDebug("refresh = ${refresh}")
```

Also consider auto-disabling `logEnable` after 30 minutes (standard Hubitat pattern):

```groovy
def updated() {
    if (logEnable) runIn(1800, "logsOff")
    initialize()
}
def logsOff() {
    log.warn "debug logging auto-disabled"
    app.updateSetting("logEnable", [value: "false", type: "bool"])
}
```

### `ignoreSSLIssues: true` on the WebSocket

`devices/smartHQ_connector` connects with `ignoreSSLIssues: true`, which disables SSL certificate validation for the WebSocket. This was carried over from the original code; if the GE Brillion endpoint has a valid certificate (which it should, given it's a public AWS endpoint), this flag should be removed to restore MITM protection.

### Lock capability on Laundry

The laundry driver has a `doorLock` attribute (locked/unlocked) but no `Lock` capability declaration. If the SmartHQ API supports remote door-lock control (likely yes for some models), wire up `lock()` / `unlock()` commands and add the capability. If not, leave as-is — the attribute is informational only.

### ThermostatMode constraints on Portable AC

The Portable AC's `setThermostatMode` is constrained to AC operating modes (cool, fan_only, energy_saver, dry, default). Standard Hubitat ThermostatMode dashboards expect (off, cool, heat, auto, emergency heat). Some dashboards may not render the AC's custom modes in their mode picker. Worth verifying once installed in a real dashboard and either leaving as-is or aliasing the modes.

### Microwave on()/off() implementation

Microwave `on()` is currently a no-op log warning. The driver does have `adjustMicrowaveState("on" / "off")` which can technically start/stop the microwave (subject to the appliance accepting the command). Consider wiring `off()` → `adjustMicrowaveState("off")` for at least the stop case (starting remotely is a safety concern; stopping isn't).
