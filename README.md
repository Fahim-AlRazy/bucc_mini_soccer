# Alianza Bots Arcade: 3D Mini Soccer

A responsive, high-performance 3D football arcade game built for local PvP (1v1) multiplayer and single-player VS AI play. The project is fully self-contained, optimized for showcase booths, and runs locally without server requirements.

---

## 🚀 Tech Stack

1. **3D Render Engine**: [Three.js (r128)](https://threejs.org/) — renders all 3D geometries, materials, field markings, stadium lighting, and player meshes.
2. **User Interface (UI/HUD)**: Vanilla HTML5 and CSS3 — manages game menus, neon-glow input fields, scoreboards, and pause configurations.
3. **Sound System**: HTML5 Audio & Web Audio API — handles custom local sound effect playback and synthesizes fallback whistle waveforms.
4. **Typography**: [Google Fonts (Outfit)](https://fonts.google.com/specimen/Outfit) — premium broadcast-style UI font.

---

## ⚙️ How It Works & Core Mechanisms

### 1. Game & Physics Engine
* **Collision Detection**: 2D circle-to-circle physics handles collisions between all player models and the ball. If circles overlap, the engine calculates the normal vector of collision and pushes the meshes apart to resolve the overlap.
* **Friction & Boundaries**: A constant friction factor (`0.98`) is applied to the ball's velocity vector frame-by-frame. Boundaries (walls) and goalposts clamp velocities and reflect ball trajectory cleanly.
* **Possession & Dribbling**: When a player contacts the ball (outside of tackle cooldowns), they gain possession. The ball coordinates are snapped to an offset position in front of the possessing bot's feet, and visual spin is calculated dynamically based on movement velocity.
* **Tackling / Snatching**: If a defender approaches a player possessing the ball from the front or side, the defender "tackles". The ball is instantly released, kicked in the defender's facing direction, and cooldown timers (30 frames) are set on both bots to prevent immediate repossession.

### 2. Smart AI Strategy
* **AI Defender**: Targets the ball when defending, tackles to steal possession, and transitions to offensive chaser mode once in possession.
* **Corner Shooting**: When the AI possessor enters the offensive third, it target-shoots toward the goal corners furthest from the goalkeeper (`Z = -3.5` or `Z = 3.5`), maximizing scoring efficiency.
* **AI Goalkeepers**: Move dynamically along the goal line, tracking the ball's horizontal position, and dynamically rotate to face the ball. Goalkeeper speed is balanced at `0.11` to challenge shooters.

### 3. Visuals & Theme
* **Arena Design**: The field features a high-density green turf pattern, stadium light towers, 3D flags, and advertising boards displaying the **BUCC (BRAC University Computer Club)** logo.
* **Colored Goal Floors**: The floors under the goal nets are styled with colored planes matching the advertisement theme (Gold/Yellow for the left goalmouth, Blue for the right goalmouth) placed at `y = 0.005` to avoid z-fighting.

---

## 🎮 Keyboard Controls & Player Separation

To enable 1v1 local PvP gameplay on a single keyboard, the system uses two separate mappings and handles inputs as follows:

### 1. Player Mappings
* **Player 1 (Left Side / Yellow)**:
  * **Movement**: `W`, `A`, `S`, `D` (or Arrow Keys)
  * **Kick / Pass**: `Spacebar`
* **Player 2 (Right Side / Blue)**:
  * **Movement**: Numpad `8` (Up), `5` (Down), `4` (Left), `6` (Right)
  * **Kick / Pass**: Numpad `0` (or `Insert` / digit `0` for compact laptop layouts)

### 2. Handling Key Listener States
A global keyboard registry object (`keys`) stores active boolean values for each control register:
```javascript
const keys = {
    w: false, a: false, s: false, d: false,
    arrowup: false, arrowdown: false, arrowleft: false, arrowright: false,
    space: false,
    numpadUp: false, numpadDown: false, numpadLeft: false, numpadRight: false,
    numpadKick: false
};
```
* **Event Listeners**: Keydown and keyup events toggle the registry flags.
* **Double-Map Key Code & Key Character**: Listening to both `e.code` (e.g. `'Numpad8'`) and `e.key` (e.g. `'8'`) ensures Player 2 controls function regardless of NumLock state or layout.
* **Prevent Default Scrolling**: Handlers invoke `e.preventDefault()` on key registers (like `Space` and `Numpad0`) to prevent webpage scroll jumps.

### 3. Typing vs. Gameplay Input Separation
To prevent name input fields from swallowing keys or triggering player movement during rules configuration, key listeners check the active DOM element:
```javascript
window.addEventListener('keydown', (e) => {
    if (document.activeElement && document.activeElement.tagName === 'INPUT') {
        return; // Allows players to type spaces/numbers inside name fields
    }
    // Gameplay key registers...
});
```
When the game starts, `document.activeElement.blur()` is called automatically to return keyboard focus back to the game canvas.

---

## 🎵 CORS-Free Audio Architecture

Modern web browsers block standard HTTP `fetch()` requests targeting the local filesystem (`file://` protocol) due to Cross-Origin Resource Sharing (CORS) security guidelines. To make this game fully portable (i.e. runnable by double-clicking `index.html` from a USB drive or local folder):

1. **Preloaded Audio Tags**: Standard HTML5 audio elements are embedded directly in the HTML `<body>`:
   ```html
   <audio id="sound-start" src="sound/start_sound.mp3" preload="auto"></audio>
   <audio id="sound-goal" src="sound/goal_sound.mp3" preload="auto"></audio>
   <audio id="sound-end" src="sound/end_sound.mp3" preload="auto"></audio>
   ```
2. **Element Playback**: The `SoundEffects` controller plays the media by calling `audioEl.play()`:
   ```javascript
   playAudioElement(key) {
       const audioEl = document.getElementById('sound-' + key);
       if (audioEl) {
           audioEl.currentTime = 0;
           audioEl.play().catch(err => {
               // Fallback whistle synth triggers if browser blocks play
           });
       }
   }
   ```
This setup runs without server environments, bypassing CORS issues while providing custom MP3 playback.

---

## 🕹️ Instructions to Run

1. Open the folder containing the project files.
2. Double-click the `index.html` file to launch the game directly in any modern browser.
3. Select your game mode, input custom names, and click **KICK OFF** to play.
