

## Overview

- **Short Press:**  
    A brief tap toggles the CapsLock state (e.g., turning CapsLock on or off) just like usual.
    
- **Long Press (e.g., ≥300 ms):**  
    Holding the key for longer than the threshold (set here to 300 ms) triggers the hyper modifier mode. In this mode, the script sends the modifier keys (**LWin, Ctrl, Alt, Shift**) down and holds them until the CapsLock key is released.
    
- **Purpose of the Threshold:**  
    Adjusting the threshold (in this example, 300 ms) provides a balance:
    
    - **Too short (e.g., 200 ms):** May trigger the hyper modifier unintentionally.
    - **Too long (e.g., >400 ms):** May delay the modifier response when it is actually desired.
    
    The 300 ms delay strikes a good balance between responsiveness and avoiding accidental modifier mode activation.
    

---

## The Script

```ahk
#Requires AutoHotkey v2.0


CapsLock::
{
    static modifierMode := false  ; Make it static so it persists between function calls
    pressStartTime := A_TickCount
    modifierSent := false
    
    ; Wait for either 2 seconds or CapsLock release
    while (GetKeyState("CapsLock", "P")) {
        if (A_TickCount - pressStartTime >= 300 && !modifierSent) {
            ; Enter modifier mode and send modifier keys
            modifierMode := true
            Send "{LWin Down}{Ctrl Down}{Alt Down}{Shift Down}"
            modifierSent := true
        }
        Sleep 10
    }
    
    ; If we entered modifier mode
    if (modifierMode) {
        ; Release all modifier keys
        Send "{LWin Up}{Ctrl Up}{Alt Up}{Shift Up}"
        modifierMode := false
    } else {
        ; If CapsLock was pressed briefly, toggle CapsLock state
        SetCapsLockState !GetKeyState("CapsLock", "T")
    }
}

; Block native CapsLock behavior to prevent double-toggling
*CapsLock::Return
```

## Explanation of the Script

1. **Version Requirement:**
    
    - `#Requires AutoHotkey v2.0`  
        This line ensures that the script runs only in AutoHotkey v2, as the syntax and functions differ from version 1.
2. **Hotkey Definition and Variables:**
    
    - The hotkey `CapsLock::` starts the custom behavior.
    - `static modifierMode := false`  
        Declares a static variable that remembers if the hyper modifier has been activated.
    - `pressStartTime := A_TickCount`  
        Captures the time when CapsLock is pressed to measure the hold duration.
    - `modifierSent := false`  
        A flag to make sure the modifier keys are sent only once per key press.
3. **Monitoring the Press Duration:**
    
    - The `while (GetKeyState("CapsLock", "P"))` loop keeps checking if the key is still pressed.
    - Inside the loop:
        - The `if` statement checks if the elapsed time (`A_TickCount - pressStartTime`) is at least **300 ms** and if the modifier has not yet been sent.
        - If the conditions are met, the script enters modifier mode by:
            - Setting `modifierMode` to `true`.
            - Sending the modifier key-down events (`LWin`, `Ctrl`, `Alt`, and `Shift`).
            - Marking `modifierSent` as `true` to avoid sending the keys repeatedly.
        - `Sleep 10` helps in reducing CPU usage by pausing briefly between state checks.
4. **Handling the Key Release:**
    
    - Once the key is released, the script exits the loop.
    - If `modifierMode` is `true`, the script sends the key-up events for all modifiers and resets `modifierMode`.
    - If not, it toggles the CapsLock state by using `SetCapsLockState !GetKeyState("CapsLock", "T")`. This means a quick tap will behave like a normal CapsLock toggle.
5. **Blocking Native Behavior:**
    
    - `*CapsLock::Return` prevents the system's native CapsLock functionality from triggering alongside your script, ensuring that only the intended behavior occurs.

