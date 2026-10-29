# Home Assistant Blueprint: Alarmo Calendar Access

Automatically disarm [Alarmo](https://github.com/nielsfaber/alarmo) ahead of scheduled calendar events, such as cleaning appointments or planned visits, and receive reminders when those events end.

The blueprint uses Home Assistant's Calendar integration and supports Google Calendar events marked with a configurable text string.

**Safety principle:** This blueprint can disarm the alarm, but it will never arm it automatically. When an event ends, it reminds you to arm the alarm if you are away.

## Features

- **Scheduled disarming:** Disarm Alarmo a configurable amount of time before a matching calendar event.
- **Event matching:** Match a configurable marker in the event title or description, ignoring capitalization.
- **Mobile notifications:** Notify your phone when the alarm is disarmed and when the event ends.
- **Persistent notifications:** Create notifications in the Home Assistant interface for disarming and event-end reminders.
- **Independent notification controls:** Enable or disable mobile notifications, the disarm notification, and the event-end notification separately.
- **Startup recovery:** Check for matching events after Home Assistant starts and attempt to recover a missed disarm.
- **No automatic arming:** The blueprint never arms the alarm, regardless of event status.

## Requirements

- Home Assistant with the Calendar integration configured.
- A calendar entity containing the relevant events, such as Google Calendar.
- [Alarmo](https://github.com/nielsfaber/alarmo) configured as your alarm control panel.
- A mobile notification service if mobile notifications are enabled.

## Installation

### Option 1: Import the blueprint manually

1. Create a directory for your blueprints if you do not already have one:

   ```text
   config/blueprints/automation/<your_name>/
   ```

2. Save the blueprint YAML file in that directory, for example:

   ```text
   config/blueprints/automation/<your_name>/alarmo_calendar_access.yaml
   ```

3. In Home Assistant, open **Settings → Automations & scenes → Blueprints**.
4. Reload the automations/blueprints if necessary, or restart Home Assistant.
5. Open the blueprint and select **Create automation**.

### Option 2: Install from GitHub

Once this project is published, you can distribute the YAML file through your repository and link directly to the raw blueprint file. You can also add a Home Assistant blueprint import link to the repository once the GitHub repository URL is known.

## Configuration

When creating an automation from the blueprint, configure the following inputs.

| Input | Description | Default |
|---|---|---|
| Google Calendar | Calendar containing the relevant events | Required |
| Alarm panel | Alarmo alarm control panel entity | Required |
| Mobile notification service | Notification service, e.g. `notify.mobile_app_your_phone` | Required |
| Event marker | Text that identifies events requiring disarming | `Alarmo Off` |
| Disarm before event | Time before the event starts to disarm | `-00:30:00` |
| Enable mobile notifications | Enable phone notifications at event start and end | Enabled |
| Persistent notification when disarmed | Create a Home Assistant notification when disarming | Enabled |
| Persistent notification when event ends | Create a Home Assistant reminder when the event ends | Enabled |

### Mobile notification service

Enter the notification service exposed by your Home Assistant installation, for example:

```yaml
notify.mobile_app_your_phone
```

Replace this example with the actual service name available on your system.

## Marking calendar events

By default, an event qualifies when its title or description contains `Alarmo Off`.

Examples of qualifying events:

**Event title:**

```text
House cleaning — Alarmo Off
```

**Event description:**

```text
Regular cleaning appointment.
Alarmo Off
```

Matching is case-insensitive, so `alarmo off`, `Alarmo Off`, and `ALARMO OFF` all qualify.

Events without the configured marker are ignored.

The marker is a simple text match. It does not require the marker to be a separate word or line.

## How it works

### 1. Before an event starts

The blueprint listens for the calendar's start trigger with the configured negative offset.

For example, with a 30-minute offset:

| Time | Action |
|---|---|
| 09:30 | Alarmo is disarmed |
| 10:00 | The calendar event begins |

If enabled, the blueprint sends a mobile notification and/or creates a persistent Home Assistant notification explaining that the alarm was disarmed for the event.

### 2. When an event ends

The blueprint listens for the calendar's end trigger.

If enabled, it sends a mobile notification and/or creates a persistent Home Assistant notification reminding you to arm Alarmo if you are away.

**The alarm is not armed automatically when the event ends.**

### 3. Startup recovery

If Home Assistant was offline when the scheduled disarm should have happened, the blueprint attempts to recover when Home Assistant starts.

The recovery logic queries the configured calendar and looks for events that:

- Contain the configured marker in their title or description.
- Have already reached their scheduled disarm time.
- Have not yet ended.

If a qualifying event is found, the blueprint disarms Alarmo and sends the enabled disarm notifications.

For example, if an event starts at 10:00 and the disarm offset is 30 minutes, but Home Assistant restarts at 09:40 after missing the 09:30 trigger, the recovery logic attempts to disarm Alarmo.

Recovery depends on the calendar integration being available and returning the relevant event data when Home Assistant starts.

## Notification behavior

The three notification options are independent:

| Mobile notifications | Persistent disarm | Persistent event-end | Result |
|---|---|---|---|
| Enabled | Enabled | Enabled | All notifications |
| Disabled | Enabled | Enabled | Home Assistant notifications only |
| Enabled | Disabled | Disabled | Mobile notifications only |
| Disabled | Disabled | Enabled | Persistent event-end reminders only |

Persistent notifications use fixed IDs for each notification type. A subsequent notification of the same type replaces the previous one rather than creating another persistent notification with the same ID.

## Limitations and safety considerations

- **Calendar availability:** The blueprint depends on Home Assistant receiving calendar events and being able to query the calendar during startup recovery.
- **Missed triggers:** Startup recovery attempts to compensate for downtime, but it cannot guarantee recovery if the calendar is unavailable or event data is missing.
- **No recovery after an event ends:** The startup recovery check does not disarm for events that have already ended.
- **No automatic arming:** The blueprint intentionally does not arm the alarm at the end of an event. You remain responsible for securing the house.
- **Test before relying on it:** Verify the normal disarm trigger, startup recovery, event-end reminders, and notification settings with a test event before using the blueprint for real appointments.

This blueprint is a convenience automation, not a substitute for verifying the alarm's state before leaving home.

## Troubleshooting

### The alarm was not disarmed

Check that:

1. The event title or description contains the configured marker.
2. The selected calendar entity is correct.
3. The configured offset is valid and negative.
4. Home Assistant was running at the scheduled trigger time, or the startup recovery query successfully retrieved the event.
5. The automation trace shows that the correct trigger and action ran.

### Notifications are not received

Check that:

1. The mobile notification service name is correct.
2. Mobile notifications are enabled in the automation.
3. The relevant persistent notification option is enabled if you expect a notification inside Home Assistant.
4. The automation trace contains the notification action.

### The event-end reminder was not sent

Check that the event has an end time, the calendar integration delivers end triggers, and the automation trace shows the `access_end` trigger.

## License

Choose and add a license file before publishing this project. If you want others to reuse and modify the blueprint, a permissive license such as MIT may be suitable.
