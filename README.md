# Speed Dial — WxCC Desktop Consult Buttons

A Webex Contact Center (WxCC) Agent Desktop widget that adds up to three one-click **speed dial consult buttons** to the interaction panel. During a voice call, the agent clicks a button, such as "Nurse Line" or "Supervisor", and the widget starts a consult to that number. The agent doesn't have to open the consult dialog and type or look up the destination.

<!-- Optional: add a screenshot, e.g. ![Speed dial buttons on an active call](screenshot.png) -->

## What it does

- Shows up to **3 buttons**. Each one has a label and a dial number (DN) that you set in the Desktop Layout.
- The buttons **only appear while the agent is on a call**, and they enable and disable as the call changes:

| Call state | Buttons |
|------------|---------|
| No call / wrap-up / ended | Hidden |
| Connected or in a conference | Shown and clickable |
| Consult in progress | Shown but disabled |

- Clicking a button starts a **consult to that DN** on the agent's current call. The agent can then transfer or conference from the normal desktop controls.
- **Double-click protection:**
  - All buttons disable as soon as one is clicked.
  - Any click within **5 seconds** of the last one is ignored.
- A button is left out completely if its name or DN is blank. You can use one, two or three.

## How it works

1. **Startup.** On load, the widget initializes the [WxCC Desktop JS SDK](https://developer.webex-cx.com/documentation/guides/desktop) (`@wxcc-desktop/sdk`, bundled in) and subscribes to the agent contact events: offer, consult, conference, hold, wrap-up, ended, and others.
2. **Tracking the call state.** Each time an event fires, the widget works out the call state from the event's interaction data:
   - It reads the interaction's `state`, `owner` and `isTerminated` flag.
   - It checks the event type (e.g. `AgentConsultCreated` → consulting, `AgentWrapup` → wrap-up).
   - It checks whether any participant still on the call has a `consultState` of `consulting` or `consultInitiated`.
   - It treats a connected call whose `relationshipType` is `consult` as a consult in progress.

   It then shows, hides, enables or disables the buttons to match.
3. **Placing the consult.** When the agent clicks a button, the widget calls:

   ```js
   Desktop.agentContact.consult({
     interactionId,                 // the agent's current task
     data: { destinationType: "DN", destAgentId: "<buttonN_dn>" }
   })
   ```

4. **Logging.** Everything is logged through the Desktop SDK logger under the name `cisco-conference-speed-dial`. You can see the logs in the browser console or in the agent's downloaded desktop logs.

## Requirements

- Webex Contact Center Agent Desktop. Works in any region, because it uses the Desktop SDK and no hard-coded API URLs.
- An Agent Desktop layout you can edit and upload in Control Hub.
- Agents who are allowed to consult to a DN, and dial numbers your dial plan can reach.
- A host for `index.js` that the desktop can reach over HTTPS.

## Installation

### 1. Host the script

The script is served from this repo by GitHub Pages:

```
https://krich5.github.io/SpeedDial/index.js
```

To host your own copy, fork this repo and turn on GitHub Pages, or upload `index.js` to any static HTTPS host.

### 2. Add it to your Desktop Layout

The component name is `conference-control-ui`. Place it in the interaction panel's control area so the buttons appear next to the call controls. Pass in the button labels and DNs, plus the agent's task context from `$STORE`:

```json
{
  "comp": "conference-control-ui",
  "script": "https://krich5.github.io/SpeedDial/index.js",
  "properties": {
    "button1_name": "Nurse Line",
    "button1_dn": "+15555550101",
    "button2_name": "Supervisor",
    "button2_dn": "+15555550102",
    "button3_name": "Billing",
    "button3_dn": "+15555550103",
    "details": "$STORE.agentContact.taskMap",
    "selected_task": "$STORE.agentContact.taskSelected",
    "agent_id": "$STORE.agent.agentId"
  }
}
```

> A complete sample layout is included in this repo: [`Desktop_Layout_SpeedDial.json`](Desktop_Layout_SpeedDial.json)
<!-- Update the filename and the snippet above to match the layout file you upload -->

### 3. Upload the layout

In **Control Hub → Contact Center → Desktop Layouts**, upload the layout and assign it to the team(s) that should get the buttons. Agents must sign out and back in to pick up the new layout.

## Properties

| Property | Required | Description |
|----------|----------|-------------|
| `button1_name` / `button1_dn` | Optional | Label and dial number for button 1 |
| `button2_name` / `button2_dn` | Optional | Label and dial number for button 2 |
| `button3_name` / `button3_dn` | Optional | Label and dial number for button 3 |
| `details` | Yes | The agent's task map (`$STORE.agentContact.taskMap`), used to read the call state when the widget first loads |
| `selected_task` | No | The currently selected task. Only written to debug logs |
| `agent_id` | No | The agent's ID. Only written to debug logs |

A button only renders when **both** its `_name` and `_dn` are set.

## Built with

- Plain Web Components (`HTMLElement` + Shadow DOM). No framework.
- [`@wxcc-desktop/sdk`](https://www.npmjs.com/package/@wxcc-desktop/sdk), bundled with webpack
- [Momentum UI](https://momentum.design) `md-button` / `md-tooltip` / `md-icon` elements, which the Agent Desktop already provides

## Known limitations

- **Three buttons maximum.** The button slots are fixed in code.
- **Consult only, no blind transfer.** The buttons always start a consult. The agent finishes the transfer or conference from the normal call controls.
- **Only the first task is used.** If the agent has more than one task, the consult targets the first task in the task map, which may not be the call they're looking at.
- **"Hold participants" can't be configured.** The code reads a `holdParticipants` setting but then always forces it to `true`, and the value is never sent with the consult.
- **Buttons can be added twice.** If the desktop removes and re-adds the widget, `render()` adds a second copy of the buttons instead of replacing the first.
- **Button labels aren't escaped.** Labels from the layout are inserted into the page as raw HTML. Keep button names to plain text.

## License

See [LICENSE](LICENSE).
