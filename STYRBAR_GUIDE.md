# IKEA STYRBAR Remote Blueprint - Installation & Usage Guide

## Firmware Compatibility
This blueprint is specifically designed for IKEA STYRBAR (E2001/E2002) with **firmware 2.4.16** that has quirk issues causing spurious events.

## Installation Steps

### Step 1: Create the Required Helper

Before installing the blueprint, you need to create a text helper:

1. Go to **Settings** → **Devices & Services** → **Helpers**
2. Click **"+ CREATE HELPER"**
3. Select **"Text"**
4. Configure:
   - **Name**: `STYRBAR Last Button` (or any name you prefer)
   - **Icon**: `mdi:remote` (optional)
   - **Maximum length**: `20`
5. Click **"CREATE"**

### Step 2: Install the Blueprint

1. Copy the entire contents of `styrbar_blueprint.yaml`
2. Go to **Settings** → **Automations & Scenes** → **Blueprints**
3. Click **"IMPORT BLUEPRINT"** (bottom right)
4. Click **"IMPORT BLUEPRINT"** again
5. In the URL field, if you have the file hosted, paste the URL
   
   **OR**
   
   Click the three dots menu → **"Create new blueprint"** → Paste the YAML content directly

### Step 3: Create an Automation from the Blueprint

1. Go to **Settings** → **Automations & Scenes** → **Automations**
2. Click **"+ CREATE AUTOMATION"**
3. Select **"IKEA STYRBAR Remote (E2001/E2002) - Firmware 2.4.16 Fixed"**
4. Configure:
   - **STYRBAR Remote**: Select your remote device
   - **Helper - Last Button Tracker**: Select the text helper you created
   - Configure actions for each button (see examples below)

## Button Actions Available

The blueprint provides 12 configurable actions:

### ON Button (Top/Large Sun)
- **Short Press** - Single tap
- **Long Press** - Triggers when held down
- **Release** - Triggers when released after holding

### OFF Button (Bottom/Small Sun)
- **Short Press** - Single tap
- **Long Press** - Triggers when held down
- **Release** - Triggers when released after holding

### LEFT Button (Left Arrow)
- **Short Press** - Single tap
- **Long Press** - Triggers when held down (slight delay due to firmware)
- **Release** - Triggers when released after holding

### RIGHT Button (Right Arrow)
- **Short Press** - Single tap
- **Long Press** - Triggers when held down (slight delay due to firmware)
- **Release** - Triggers when released after holding

## Example Usage Scenarios

### Example 1: Basic Light Control

**ON Button:**
- **Short Press**: Turn on light at 100%
  ```yaml
  - service: light.turn_on
    target:
      entity_id: light.living_room
    data:
      brightness_pct: 100
  ```
- **Long Press**: Start dimming up
  ```yaml
  - repeat:
      while:
        - condition: state
          entity_id: input_text.styrbar_last_button
          state: "on_holding"
      sequence:
        - service: light.turn_on
          target:
            entity_id: light.living_room
          data:
            brightness_step_pct: 5
        - delay:
            milliseconds: 200
  ```

**OFF Button:**
- **Short Press**: Turn off light
  ```yaml
  - service: light.turn_off
    target:
      entity_id: light.living_room
  ```

**LEFT Button:**
- **Short Press**: Decrease color temperature (warmer)
  ```yaml
  - service: light.turn_on
    target:
      entity_id: light.living_room
    data:
      kelvin: "{{ state_attr('light.living_room', 'color_temp_kelvin') | int - 500 }}"
  ```

**RIGHT Button:**
- **Short Press**: Increase color temperature (cooler)
  ```yaml
  - service: light.turn_on
    target:
      entity_id: light.living_room
    data:
      kelvin: "{{ state_attr('light.living_room', 'color_temp_kelvin') | int + 500 }}"
  ```

### Example 2: Scene Control

**ON Button - Short Press**: Activate "Bright" scene
```yaml
- service: scene.turn_on
  target:
    entity_id: scene.living_room_bright
```

**OFF Button - Short Press**: Activate "Dim" scene
```yaml
- service: scene.turn_on
  target:
    entity_id: scene.living_room_dim
```

**LEFT Button - Short Press**: Previous scene
```yaml
- service: script.previous_scene
```

**RIGHT Button - Short Press**: Next scene
```yaml
- service: script.next_scene
```

### Example 3: Media Control

**ON Button:**
- **Short Press**: Play/Pause
  ```yaml
  - service: media_player.media_play_pause
    target:
      entity_id: media_player.living_room_tv
  ```

**OFF Button:**
- **Short Press**: Stop
  ```yaml
  - service: media_player.media_stop
    target:
      entity_id: media_player.living_room_tv
  ```

**LEFT Button:**
- **Short Press**: Previous track
  ```yaml
  - service: media_player.media_previous_track
    target:
      entity_id: media_player.living_room_tv
  ```

**RIGHT Button:**
- **Short Press**: Next track
  ```yaml
  - service: media_player.media_next_track
    target:
      entity_id: media_player.living_room_tv
  ```

**ON/OFF Long Press:**
- **Long Press**: Volume up/down continuously
  ```yaml
  # ON button long press
  - repeat:
      while:
        - condition: state
          entity_id: input_text.styrbar_last_button
          state: "on_holding"
      sequence:
        - service: media_player.volume_up
          target:
            entity_id: media_player.living_room_tv
        - delay:
            milliseconds: 300
  ```

### Example 4: Multi-Light Control

**ON Button - Short Press**: Turn on all lights
```yaml
- service: light.turn_on
  target:
    entity_id:
      - light.living_room
      - light.kitchen
      - light.hallway
```

**LEFT Button - Short Press**: Cycle through lights
```yaml
- choose:
    - conditions:
        - condition: state
          entity_id: light.living_room
          state: "off"
      sequence:
        - service: light.turn_on
          target:
            entity_id: light.living_room
    - conditions:
        - condition: state
          entity_id: light.kitchen
          state: "off"
      sequence:
        - service: light.turn_on
          target:
            entity_id: light.kitchen
  default:
    - service: light.turn_on
      target:
        entity_id: light.hallway
```

### Example 5: Color Control (RGB Lights)

**LEFT Button - Short Press**: Change to previous color
```yaml
- service: light.turn_on
  target:
    entity_id: light.rgb_bulb
  data:
    rgb_color:
      - "{{ (state_attr('light.rgb_bulb', 'rgb_color')[0] - 30) % 256 }}"
      - "{{ state_attr('light.rgb_bulb', 'rgb_color')[1] }}"
      - "{{ state_attr('light.rgb_bulb', 'rgb_color')[2] }}"
```

**RIGHT Button - Short Press**: Change to next color
```yaml
- service: light.turn_on
  target:
    entity_id: light.rgb_bulb
  data:
    rgb_color:
      - "{{ (state_attr('light.rgb_bulb', 'rgb_color')[0] + 30) % 256 }}"
      - "{{ state_attr('light.rgb_bulb', 'rgb_color')[1] }}"
      - "{{ state_attr('light.rgb_bulb', 'rgb_color')[2] }}"
```

## Tips & Best Practices

### 1. Testing Actions
Use simple notification actions first to verify buttons work:
```yaml
- service: notify.mobile_app_your_phone
  data:
    message: "Button pressed!"
```

### 2. Continuous Actions During Hold
For smooth dimming or volume control, use a repeat loop with the helper state:
```yaml
- repeat:
    while:
      - condition: state
        entity_id: input_text.styrbar_last_button
        state: "on_holding"  # or "off_holding", "left_holding", "right_holding"
    sequence:
      - service: light.turn_on
        data:
          brightness_step_pct: 5
      - delay:
          milliseconds: 200
```

### 3. Multiple Remotes
Create a separate helper for each remote:
- `styrbar_living_room_last_button`
- `styrbar_bedroom_last_button`
- etc.

Then create a separate automation from the blueprint for each remote.

### 4. Debugging
To see what's happening:
1. Go to **Developer Tools** → **States**
2. Find your helper entity (e.g., `input_text.styrbar_last_button`)
3. Watch the value change as you press buttons

### 5. Empty Actions
You don't need to configure all 12 actions. Leave any unused actions empty (default: `[]`).

## Troubleshooting

### Issue: Actions not triggering
- Verify the helper entity exists and is spelled correctly
- Check that your STYRBAR device is selected correctly
- Enable automation traces to see what's happening

### Issue: Wrong button triggering
- Check **Developer Tools** → **Events** → Listen to `zha_event`
- Press buttons and verify the command/args match the blueprint logic

### Issue: Spurious "on" commands still triggering
- This means the helper isn't tracking state properly
- Verify the helper is being set correctly by checking its state

### Issue: Left/Right buttons not distinguishing
- This is expected during the initial hold sequence (release 0, on, recall)
- The actual distinction happens when the `hold` command arrives
- The blueprint handles this automatically

## Technical Notes

### How It Works
1. **Helper Tracking**: The text helper stores which button was last pressed/held
2. **Spurious Event Filtering**: The blueprint filters out:
   - "release 0" events (phantom releases)
   - "on" events that occur during left/right holds
   - "recall" events from scene cluster
3. **State Machine**: Button states tracked:
   - `on`, `on_holding`
   - `off`, `off_holding`
   - `left`, `left_holding`
   - `right`, `right_holding`

### Event Sequences (For Reference)

**Left/Right Short Press:**
```
command: "press", args: [257, 13, 0] or [256, 13, 0]
```

**Left/Right Hold:**
```
1. command: "release", args: [0]           ← Spurious, filtered
2. command: "on"                           ← Spurious, filtered
3. command: "recall"                       ← Spurious, ignored
4. command: "hold", args: [3329, 0] or [3328, 0]  ← Real hold detected
```

**Left/Right Release:**
```
command: "release", args: [non-zero value]
```

## Support & Contribution

This blueprint was created specifically for IKEA STYRBAR firmware 2.4.16 with ZHA quirk issues.

If you encounter issues or have improvements, please share your findings!
