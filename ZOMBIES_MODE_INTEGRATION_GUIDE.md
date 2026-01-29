# Zombies Mode Integration Guide

This guide explains how to add the new **Zombies/Undead Siege mode** to the Gameloop CODM Keymap Generator.

## Overview

The zombies mode has been created as **ModeID="6"** with the name **"Zombies"**. It includes:
- All standard Multi Player keybinds
- **F key** with zombie-specific operations (开启功能（僵尸模式）)
- **Right Click** with zombie grenade cancellation (取消僵尸雷)
- **B key** with zombie axe skill (近战武器技能（僵尸斧子）)

## Step 1: Update DefaultKeyMap.xml

1. Open `composeApp/src/commonMain/composeResources/files/DefaultKeyMap.xml`
2. Find the end of the **OB mode** (ModeID="5") - look for its closing `</KeyMapMode>` tag
3. **Before the final `</Item>` tag**, insert the entire contents of `ZombiesMode.xml`

The structure should look like:

```xml
<Item ApkName="com.activision.callofduty.shooter" ...>
    <!-- Switch definitions -->
    ...
    
    <!-- ModeID 1: Multi Player -->
    <KeyMapMode ModeID="1" Name="Multi Player" ...>
        ...
    </KeyMapMode>
    
    <!-- ModeID 2: Battle Royale -->
    <KeyMapMode ModeID="2" Name="Battle Royale" ...>
        ...
    </KeyMapMode>
    
    <!-- ModeID 3: Gundam -->
    <KeyMapMode ModeID="3" Name="Gundam" ...>
        ...
    </KeyMapMode>
    
    <!-- ModeID 4: DMZ -->
    <KeyMapMode ModeID="4" Name="DMZ" ...>
        ...
    </KeyMapMode>
    
    <!-- ModeID 5: OB -->
    <KeyMapMode ModeID="5" Name="OB" ...>
        ...
    </KeyMapMode>
    
    <!-- ModeID 6: Zombies (NEW!) -->
    <KeyMapMode ModeID="6" Name="Zombies" ...>
        ...
    </KeyMapMode>
    
</Item>
```

## Step 2: Update ManageFile.kt

The app needs to know about the new mode for parsing and generating keymaps.

### 2.1: Add Zombies Mode Constant

In `composeApp/src/wasmJsMain/kotlin/org/napharcos/gameloopcodmkeymap/ManageFile.kt`, add:

```kotlin
// Add this with the other mode start constants
const val ZOMBIES_START = """<KeyMapMode ModeID="6" Name="Zombies" CurrModeEnableTips=""""
```

### 2.2: Update applyMode Function

Find the `applyMode` function and add the Zombies case:

```kotlin
fun applyMode(text: String, mode: Int): String {
    return when (mode) {
        1 -> replaceKeys(text, MP_START)
        2 -> replaceKeys(text, BR_START)
        3 -> replaceKeys(text, GD_START)
        4 -> replaceKeys(text, DMZ_START)
        5 -> replaceKeys(text, OB_START)  // If this exists
        6 -> replaceKeys(text, ZOMBIES_START)  // Add this line
        else -> text
    }
}
```

## Step 3: Update UiState.kt (if needed)

If the app has mode selection UI, add Zombies mode to the list:

```kotlin
enum class GameMode(val id: Int, val displayName: String) {
    MULTI_PLAYER(1, "Multi Player"),
    BATTLE_ROYALE(2, "Battle Royale"),
    GUNDAM(3, "Gundam"),
    DMZ(4, "DMZ"),
    OB(5, "OB"),
    ZOMBIES(6, "Zombies")  // Add this
}
```

## Step 4: Update UI Components

### 4.1: Add Zombies Option to Mode Selector

In `MainScreen.kt` (or wherever mode selection UI is), add a button/option for Zombies mode:

```kotlin
// Example with existing mode selection
ModeButton(
    text = "Zombies",
    selected = currentMode == 6,
    onClick = { onModeChange(6) }
)
```

### 4.2: Update String Resources (Optional)

Add localized strings for the Zombies mode in:
- `composeApp/src/commonMain/composeResources/values/values.xml`
- `composeApp/src/commonMain/composeResources/values-hu/values-hu.xml`

Example:
```xml
<string name="mode_zombies">Zombies</string>
<string name="mode_zombies_description">僵尸模式/不死围攻</string>
```

## Step 5: Testing

### 5.1: Local Testing

1. Build the project:
   ```bash
   ./gradlew :composeApp:wasmJsBrowserDistribution
   ```

2. Serve the built files locally and test in browser

3. Verify:
   - Zombies mode appears in the mode selector
   - Generated XML includes the new mode
   - Keybinds are correctly applied

### 5.2: In-Game Testing

1. Generate a keymap with Zombies mode selected
2. Save the XML file to Gameloop's keymap directory:
   - `C:\Users\[YourUser]\AppData\Roaming\AndroidTbox\TVM_100.xml`
3. Launch CODM in Gameloop
4. In the keybinding menu:
   - Manually switch to mode 6 (Zombies)
   - Test all keybinds, especially F key for zombie functions
   - Verify Right Click cancels zombie grenade
   - Test B key for zombie axe skill

## Key Features of Zombies Mode

### Primary Zombie Keybinds

1. **F Key (AsciiCode="70")**
   - Description: "僵尸模式功能" (Zombie Mode Functions)
   - Primary Action: "开启功能（僵尸模式）" - Open zombie mode function
   - EnableSwitch: "OpenFunction1"

2. **Right Click (AsciiCode="-2")**
   - Additional Action: "取消僵尸雷" - Cancel zombie grenade
   - EnableSwitch: "Sniper2"

3. **B Key (AsciiCode="$bCode")**
   - Description: "近战武器技能（僵尸斧子）" - Melee weapon skill (zombie axe)
   - EnableSwitch: "Hand"

### All Standard Keybinds

- WASD: Movement
- Space: Jump
- C: Crouch/Prone
- R: Reload
- G: Grenade
- Q: Tactical equipment
- 1-6: Weapon/equipment slots
- Tab: Inventory
- M: Map
- E: Interact/defuse
- X: Melee
- V: Special (tank/mech boost)
- Z: Watch teammates
- Esc: Settings

## Troubleshooting

### Mode Not Appearing
- Check that DefaultKeyMap.xml was updated correctly
- Verify ModeID="6" is unique and properly formatted
- Rebuild the project completely

### Keybinds Not Working In-Game
- Ensure Gameloop recognizes ModeID="6"
- Try testing with manual mode switching first
- Check that all `EnableSwitch` and `DisableSwitch` values are correct

### XML Parsing Errors
- Validate XML syntax (no unclosed tags)
- Ensure all `$variable` placeholders are properly replaced by the app
- Check that the structure matches other modes exactly

## File Checklist

Files that need to be modified:
- [ ] `DefaultKeyMap.xml` - Add zombies mode XML
- [ ] `ManageFile.kt` - Add ZOMBIES_START constant and applyMode case
- [ ] `UiState.kt` - Add GameMode enum value (if exists)
- [ ] `MainScreen.kt` - Add UI option for zombies mode
- [ ] String resource files - Add localized strings (optional)

## Notes

- The F key is the **primary zombie function key** with special zombie-specific operations at the top of its SwitchOperation list
- Right Click includes zombie grenade cancellation as the first operation
- B key includes zombie axe melee skill
- All other keybinds follow standard Multi Player configuration
- Manual mode switching in Gameloop allows testing without auto-detection

## Future Enhancements

Potential additions:
1. Zombie-specific HUD elements
2. Additional zombie skills mapped to unused keys
3. Custom zombie equipment slots
4. Undead Siege base building controls
5. Turret placement and management keys
