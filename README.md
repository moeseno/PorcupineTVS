# 🦔 Porcupine (Temporal Vector Steering)

**A software-defined analog joystick for digital keyboards.**

**Porcupine** (AKA Temporal Vector Steering or TVS) is a novel input parsing algorithm for 3D navigation. It extracts **124 unique directions** from a standard digital keyboard using WASD + QE, providing the smooth, sub-angular precision of an analog thumbstick without requiring any special hardware.

### 🎮 [Demo here](https://moeseno.github.io/PorcupineTVS)

## ❓ The Problem: "Binary Cancellation"

For 30 years, almost all 3D software and video games have used **Binary Cancellation**.

-   If you press **A (Left)**, your value is -1.
    
-   If you press **D (Right)**, your value is 1.
    
-   If you hold **A + D** at the same time, the engine adds them together, gets 0, and you stop moving.

Because of this limitation, standard 3D keyboard movement is restricted to a rigid **26 directions** (forward, back, up, down, and strict 45-degree diagonals). If a cinematographer or player wants to pan at a shallow 22.5-degree angle, they are forced to "tap" the keys repetitively or switch to a mouse.

## 💡 The Solution: Temporal Vector Steering (TVS)

**Porcupine** does not treat keys purely as True/False. It introduces **Time** as a variable.

By tracking the exact timestamp of when opposite keys are pressed, PorcupineTVS establishes a hierarchy:

1.  **The Older Key (The Commitment):** Dictates the primary direction (the sign).
    
2.  **The Newer Key (The Trim):** Acts as a fractional brake, pulling the vector inward by exactly 22.5 degrees (using TAN_22_5 / 0.414).
    

**Example:**  
If you hold W + D (moving Up-Right), and then press and hold A (Left):

-   Standard engines will cancel A and D, snapping your camera to pure W (Forward).
    
-   **Porcupine** knows D is older. It honors your commitment to the Right side, but uses A to "trim" your angle, resulting in a smooth, shallow drift at a 22.5° angle.

## 🧮 The Math (Why is it called Porcupine?)

When you apply TVS logic across all 3 axes (X, Y, Z), each axis gains **5 distinct states**:

1.  Full Positive (1.0)
    
2.  Full Negative (-1.0)
    
3.  Fractional Positive (0.414)  (Pos is older, Neg is newer)
    
4.  Fractional Negative (-0.414)  (Neg is older, Pos is newer)
    
5.  Neutral (0.0)
    

(5 × 5 × 5) - 1 Neutral State = 124 Unique Directions.

When you map these 124 directional vectors in 3D space, removing the fractional redundancies leaves exactly **98 razor-sharp movement vectors**. When visualized, it looks exactly like a **Porcupine**, hence the name!

## 🚀 How to Implement it

You can drop the TVS logic into any engine. Here is the core JavaScript/Three.js implementation:
```
// 1. Store your keys and their timestamps
const keys = { w:false, a:false, s:false, d:false, q:false, e:false };
const timestamps = {};

// Update timestamps on keydown
window.addEventListener('keydown', e => {
    const k = e.key.toLowerCase();
    if (keys.hasOwnProperty(k) && !keys[k]) {
        keys[k] = true;
        timestamps[k] = window.performance.now(); // Record exact time
    }
});

// 2. The TVS Core Logic
function getTVSAxisValue(posKey, negKey) {
    const pos = keys[posKey];
    const neg = keys[negKey];
    const TAN_22_5 = 0.41421356237;
    
    if (pos && neg) {
        // The OLDER key dictates the primary direction. The NEWER key acts as trim.
        if (timestamps[posKey] < timestamps[negKey]) return TAN_22_5;
        else return -TAN_22_5;
    } 
    else if (pos) return 1.0;
    else if (neg) return -1.0;
    
    return 0.0;
}

// 3. Apply to your loop
function updateMovement() {
    const x = getTVSAxisValue('d', 'a');
    const y = getTVSAxisValue('e', 'q');
    const z = getTVSAxisValue('s', 'w');
    
    // Normalize the final vector to maintain consistent speed
    const direction = new THREE.Vector3(x, y, z).normalize();
    
    // Apply direction to your camera/player...
}
```

## 🎯 Use Cases

-   **3D Staging & Cinematography:** Allows directors to perform smooth, off-angle tracking shots and dolly pans entirely on a keyboard.
    
-   **Video Games:** Gives players access to micro-adjustment angles for precise movement without needing a gamepad.

## Is this AI?

Yes, this project was realized in less than 60 minutes using  **AI Vibecoding**. 

However, the concept of  using temporal priority to create fractional vectors is my own idea. It did not come from a prompt or a pre-existing library. I literally needed such a thing for my own 3D staging engine so I decided to get AI to do the heavy lifting. LLMs were purely used for me to bring the concept into a demo in a short amount of time. Nothing more, nothing less.

## Other

If you use **Porcupine (Temporal Vector Steering)** in your project, a credit/link back to this repository is highly appreciated!
