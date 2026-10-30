# Home Assistant Blueprint: Alarmo Calendar Access

Automatically disarm [Alarmo](https://github.com/nielsfaber/alarmo) ahead of scheduled calendar events, such as cleaning appointments or planned visits.

The blueprint uses Home Assistant's Calendar integration and identifies relevant events through a configurable text marker in the event title or description.

**Safety principle:** The blueprint can disarm the alarm, but it will never arm it automatically. When an event ends, it reminds you to arm the alarm if you are away.

## Features

- **Scheduled disarming:** Disarm Alarmo a configurable amount of time before a matching calendar event.
- **Code-protected disarming:** Supply the Alarmo disarm code through a masked password input.
- **Event matching:** Match a configurable marker in the event title or description, ignoring capitalization.
- **Disarm confirmation:** Wait for Alarmo to actually reach the `disarmed` state before reporting success.
- **Failure notification:** If disarming cannot be confirmed within the configured timeout, send an explicit warning.
- **Mobile notifications:** Notify your phone when the alarm is disarmed and when the event ends.
- **Persistent notifications:** Create notifications in the Home Assistant interface for disarming and event-end reminders.
- **Independent notification controls:** Enable or disable mobile notifications, the disarm notification, and the event-end notification separately.
- **Startup recovery:** Check for matching events after Home Assistant starts and attempt to recover a missed disarm.
- **No automatic arming:** The blueprint never arms the alarm, regardless of event status.

## Requirements

- Home Assistant with the Calendar integration configured.
- A calendar entity containing the relevant events, such as Google Calendar.
- [Alarmo](https://github.com/nielsfaber/alarmo) configured as your alarm control panel.
- An Alarmo disarm code if code verification is required.
- A mobile notification service if mobile notifications are enabled.

## Installation

### Option 1: Install manually

1. Create a directory for your blueprints if you do not already have one:

   ```text
   config/blueprints/automation/<your_name>/
   ```

2. Save the blueprint YAML file in that directory, for example:

   ```text
   config/blueprints/automation/<your_name>/alarmo_calendar_access.yaml
   ```

3. In Home Assistant, open **Settings → Automations & scenes → Blueprints**.
4. Reload the blueprints if necessary, or restart Home Assistant.
5. Open the blueprint and select **Create automation**.

### Option 2: Install from GitHub

Once published, the blueprint can be imported using a Home Assistant blueprint import link pointing to the raw YAML file in the GitHub repository.

## Configuration

When creating an automation from the blueprint, configure the following inputs.

| Input | Description | Default |
|---|---|---|
| Google Calendar | Calendar containing the relevant events | Required |
| Alarm panel | Alarmo alarm control panel entity | Required |
| Alarmo disarm code | Code required to disarm Alarmo | Required |
| Mobile notification service | Notification service, e.g. `notify.mobile_app_your_phone` | Required |
| Event marker | Text identifying events requiring disarming | `Alarmo Off` |
| Disarm before event | Time before the event starts to disarm | `-00:30:00` |
| Enable mobile notifications | Phone notifications at event start and end | Enabled |
| Persistent notification when disarmed | Home Assistant notification after disarming | Enabled |
| Persistent notification when event ends | Home Assistant reminder when the event ends | Enabled |

### Alarmo disarm code

The disarm code is entered using a password-style input:

```yaml
alarm_code:
  name: Alarmo disarm code
  description: Code required by Alarmo to disarm the alarm.
  selector:
    text:
      type: password
```

The input is visually masked in the automation configuration.

**Security note:** Masking hides the characters on screen. It does not encrypt the value or guarantee protection against someone with access to your Home Assistant configuration or automation data.

### Mobile notification service

Enter the notification service exposed by your Home Assistant installation, for example:

```yaml
