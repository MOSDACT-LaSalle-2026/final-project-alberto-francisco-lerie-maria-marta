# Language Visualizer
**MDACT Project — 1st Block**

##Authors:
Alberto Benavent Ramón, Francisco A. Rodríguez. Lerie Pemanagpo, María Spínola Lasso and Marta Alavedra Marion

A real-time particle-based language visualizer that receives semantic parameters from an LLM via JSON and renders them as geometric formations in 3D space, accompanied by generative MIDI chords.

Visuals inspired by:
- *Genuary2026_16: Order and disorder* by KaitoFMS
- *260401 Particles* by Vivian

---

## Concept

Each word processed by the LLM is described by 7 semantic parameters. These parameters drive both the visual formation and the musical chord generated in real time. The central word from `input.json` is rendered as a large particle formation at the center of the canvas. Words from the poem (`poem_words.json`) fly through the 3D space around it, smaller and more transparent.

---

## File Structure

```
project/
├── final_project_block1.pde  — Main sketch: setup, draw, camera, keyboard
├── Config.pde                — Global variables and shared utilities
├── JSONLoader.pde            — Watches input.json and updates parameters
├── SymbolSystem.pde          — SymbolFormation + SymParticle (particle geometry)
├── WordsSystem.pde           — CentralWord + PoemWord (word lifecycle)
├── ClaudeSketch.pde          — LLM interface: input field, API state, drawing overlay
├── ai_agent.pde              — Anthropic API call (background thread)
├── utils.pde                 — HTTP body builder, JSON helpers, file I/O, random fallback
├── SoundEngine.pde           — MIDI chord generation entry point
├── MidiController.pde        — Low-level MIDI output via themidibus
├── ChordBuilder.pde          — Maps semantic parameters to MIDI notes
├── JSONParser.pde            — Helper to trigger sound from a raw JSON string (testing)
├── miditest.pde              — MIDI test sketch (commented out, not active)
└── data/
    ├── input.json            — Current word from LLM (watched automatically)
    └── poem_words.json       — Full poem word list with parameters
└── prompts/
    └── haiku_art_system_prompt.md  — System prompt sent to Claude with each request
```

---

## The 7 Parameters

Each word is described by these fields in the JSON:

| Parameter | Range | Visual effect | Sound effect |
|---|---|---|---|
| `emotion` | 1–7 | Base color tone | Chord key / scale |
| `word_class` | 1–8 | Geometric shape | Chord degree |
| `abstraction` | 1–7 | Formation size | Arpeggio step time |
| `agency` | 1–7 | Particle lerp speed | Note velocity |
| `organic` | 1–7 | Breathing amplitude | Chord octave |
| `phenomena_class` | 1–7 | Particle size + movement | MIDI channel (1–7) |
| `time_duration` | 1–7 | Time on screen | Note duration |
| `color_hex` | `#RRGGBB` | Exact particle color | — |

### Word Class → Shape

| Value | Class | Shape |
|---|---|---|
| 1 | Noun | Vogel spiral |
| 2 | Verb | Directional flow |
| 3 | Adjective | Open ring |
| 4 | Adverb | Expanding spiral |
| 5 | Preposition | Constellation (5 clusters) |
| 6 | Det / Pronoun | Ordered grid |
| 7 | Interjection | Radial burst (16 rays) |
| 8 | Conjunction | Filled triangle |

### Visual effect of each parameter on the formation

- **`emotion`** — sets the base color tone of all particles. Each value maps to a distinct hue: 1=red (anger), 2=yellow (surprise), 3=violet (fear), 4=pink (love), 5=green (disgust), 6=blue (sadness), 7=gold (joy).
- **`word_class`** — determines the geometric shape the particles form (see table above).
- **`abstraction`** — controls the overall size of the formation. 1=very compact at center, 7=fills most of the canvas.
- **`agency`** — controls how quickly particles travel to their target position. 1=slow, languid movement, 7=fast, energetic snapping into place.
- **`organic`** — controls how much particles oscillate and breathe around their target. 1=almost static, precise formation, 7=the shape breathes and moves continuously.
- **`phenomena_class`** — controls the size of each individual particle and applies a secondary movement character to the formation: 1=small static dots, 2=soft dispersion, 3=spiral deformation, 4=noisy sub-clusters, 5=high-frequency vibration, 6=directional drift, 7=three spatial groupings with large dots.
- **`time_duration`** — controls how long the word stays visible on screen. 1=appears briefly and fades quickly, 7=lingers for a long time.

---

## JSON Format

### `input.json` (current word)
```json
{
  "word": "Television",
  "emotion": 7,
  "emotion_label": "Joy / Happiness",
  "color_hex": "#FFA500",
  "word_class": 1,
  "abstraction": 4,
  "agency": 5,
  "organic": 2,
  "phenomena_class": 1,
  "time_duration": 5
}
```

### `poem_words.json`
```json
{
  "poem": "All Watched Over by Machines of Loving Grace",
  "author": "Richard Brautigan",
  "words": [
    {
      "word": "grace",
      "color_hex": "#FFD700",
      "word_class": 1,
      "abstraction": 6,
      "agency": 4,
      "organic": 5,
      "phenomena_class": 3,
      "time_duration": 7
    }
  ]
}
```

---

## Setup

### Requirements
- [Processing 4](https://processing.org/download)
- Library: **themidibus** (install via Processing's Library Manager)
- An Anthropic API key (optional — see LLM Interface below)

### Running
1. Open `final_project_block1.pde` in Processing
2. Make sure `input.json` and `poem_words.json` are in the `data/` folder
3. Make sure `haiku_art_system_prompt.md` is in the `prompts/` folder
4. Run the sketch — it will watch `input.json` automatically and reload on changes

### Updating the word
Type a word in the input field at the bottom of the screen and press Enter. If an API key is configured, Claude will analyse the word and generate the parameters. If not, random parameters are generated automatically.

---

## LLM Interface

The sketch includes a text input field at the bottom of the screen. Type any word and press Enter.

**With API key** — Claude analyses the word and returns the 7 parameters as JSON, which is saved to `data/input.json` and immediately applied to the visual and sound.

**Without API key** — random values are generated for all 7 parameters. Useful for testing without an Anthropic account.

To configure the API key, open `Config.pde`:
```java
final String API_KEY = "sk-ant-api03-...";  // ← your key here
```

The model and system prompt path can also be changed there:
```java
final String MODEL       = "claude-haiku-4-5-20251001";
final String prompt_path = "prompts/haiku_art_system_prompt.md";
```

---

## Camera Modes

All keyboard shortcuts require **Ctrl** (Mac and Windows).

| Key | Mode |
|---|---|
| `Ctrl+3` | Default — subtle 3D drift |
| `Ctrl+2` | Flat 2D orthographic view |
| `Ctrl+G` | Full orbit around the formation |
| `Ctrl+M` | Mouse drag — click and drag to rotate, releases to hold position |

| Key | Action |
|---|---|
| `Ctrl+R` | Force reload `input.json` |
| `Ctrl+P` | Toggle poem words visibility (show/hide background words) |

---

## Sound Engine

Controls the generation of MIDI chords for the sketch.

### Installation

1. Install the library themidibus.
2. Copy the classes SoundEngine, MidiController, ChordBuilder.
3. JSONParser is optional, used for testing.

### Usage

In the **Config.pde file**, change the value of MIDI_PORT to the name of your MIDI loopback as it is setup in your computer. (On Windows, use *Windows MIDI and Musician Settings*, as third-party loopback programs do not work as of a recent update.)

In your **main .pde file**: inside your setup function, call `setupSoundEngine()`.

Then, inside your dispose function, call `disposeSoundEngine()`.

To generate a sound based on parameters, use the function:

```java
void playWord(int emotion, int word_class, int abstraction, int agency, int organic, int phenomena_class, int time_duration);
```

Pass the parameters as received from the LLM and the chord will be sent via MIDI.

Run the Reaper project (or any MIDI application listening on channels 1 to 7) in the background to receive the MIDI and generate the sounds.
