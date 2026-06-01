# optional-skills — health

# optional-skills — health

The **health** module provides a suite of tools for physical fitness, nutrition tracking, and real-time biometric monitoring via Brain-Computer Interface (BCI) wearables. It is divided into two primary sub-modules: `fitness-nutrition` and `neuroskill-bci`.

## 1. Fitness & Nutrition

This sub-module provides exercise lookups, nutritional data retrieval, and scientific body composition calculators.

### Data Sources
- **wger API**: An open-source database for exercises.
    - *Usage Note*: Always include `language=2` (English) and `status=2` (verified) in queries.
    - *Endpoints*: `/api/v2/exercise/`, `/api/v2/exerciseinfo/{id}/`, `/api/v2/exercise/search/`.
- **USDA FoodData Central**: Provides macro and micronutrient data for over 380,000 foods.
    - *Authentication*: Uses `USDA_API_KEY`. Defaults to `DEMO_KEY` (30 req/hr).
    - *Data Mapping*: Results are standardized to **per 100g** portions.

### Core Scripts
#### `body_calc.py`
A standalone utility for physiological computations using validated formulas.
- `bmi(weight_kg, height_cm)`: Standard Quetelet index.
- `tdee(weight_kg, height_cm, age, sex, activity)`: Uses the **Mifflin-St Jeor** equation.
- `one_rep_max(weight, reps)`: Returns an average of **Epley**, **Brzycki**, and **Lombardi** formulas.
- `macros(tdee_kcal, goal)`: Calculates P/F/C splits based on `cut`, `maintain`, or `bulk` targets.
- `bodyfat(sex, neck, waist, hip, height)`: Implements the **US Navy circumference method**.

#### `nutrition_search.py`
Handles asynchronous-style lookups to the USDA API.
- `search(query, max_results)`: Queries the `Foundation` and `SR Legacy` data types.
- `display(food)`: Parses the `foodNutrients` array to extract Energy, Protein, Fat, and Carbohydrates.

---

## 2. NeuroSkill BCI Integration

The `neuroskill-bci` sub-module integrates real-time EEG and PPG data from wearables (Muse 2/S, OpenBCI) into the agent's context. It relies on the **NeuroSkill** desktop application running a local server.

### Architecture
The module communicates with the NeuroSkill server via a CLI wrapper (`npx neuroskill`) or direct HTTP/WebSocket requests to port **8375**.

```mermaid
graph TD
    A[BCI Wearable] -- BLE --> B[NeuroSkill App]
    B -- Port 8375 --> C[NeuroSkill CLI]
    C -- --json --> D[Hermes Health Module]
    D -- Context --> E[Developer/User]
```

### Key Metrics & Interpretation
Data is retrieved primarily via `npx neuroskill status --json`. Key fields in the `scores` object include:

| Metric | Logic | Threshold for Action |
| :--- | :--- | :--- |
| `focus` | $\beta / (\alpha + \theta)$ | Suggest break if < 0.40 |
| `relaxation` | $\alpha / (\beta + \theta)$ | Suggest protocol if < 0.30 |
| `cognitive_load` | Frontal $\theta$ / Temporal $\alpha$ | High load (> 0.70) suggests "Mind Dump" |
| `tbr` | Theta/Beta Ratio | Elevated (> 1.5) indicates mental fatigue |
| `faa` | Frontal Alpha Asymmetry | Negative values indicate withdrawal/low mood |

### Guided Protocols
The module includes a library of ~70 protocols in `references/protocols.md`. These are interventions triggered by specific biometric states:
- **Theta-Beta Anchor**: Triggered by high `tbr` to suppress theta and lift beta.
- **Physiological Sigh**: Triggered by acute stress spikes (high `BAR` or `stress_index`).
- **FAA Rebalancing**: Triggered by negative `faa` to shift toward positive affect.

### Historical Analysis
- **Similarity Search**: Uses `npx neuroskill search` to find neurally similar moments in history using 128-D ZUNA embeddings.
- **Sleep Staging**: `npx neuroskill sleep` provides epoch-by-epoch staging (Wake, N1, N2, N3, REM) based on AASM heuristics.
- **Comparison**: `npx neuroskill compare` calculates deltas between sessions for ~50 metrics.

## Developer Integration Patterns

### Biometric-Aware Responses
When building features that react to user state, follow this execution flow:
1. **Check Signal**: Verify `signal_quality` for all electrodes is $\ge 0.7$.
2. **Fetch Scores**: Execute `npx neuroskill status --json`.
3. **Analyze Trends**: Use `npx neuroskill session 0` to determine if metrics (like focus) are trending "up" or "down".
4. **Apply Protocol**: If a threshold is crossed, reference `protocols.md` to suggest a specific intervention.

### Nutrition & Fitness Workflows
1. **Search**: Use `nutrition_search.py` to find FDC IDs.
2. **Calculate**: Pass TDEE results from `body_calc.py` into the `macros` function to generate a meal plan.
3. **Exercise**: Query `wger` using specific `muscle` or `equipment` IDs (e.g., ID 1 for Barbell, ID 10 for Quads) to build routines.

### Error Handling
- **Port Discovery**: If the default port 8375 fails, the CLI attempts auto-discovery via mDNS.
- **Rate Limiting**: `nutrition_search.py` implements a 1-second sleep between batch queries to respect USDA `DEMO_KEY` limits.
- **Validation**: `body_calc.py` validates that `waist > neck` for male body fat calculations to prevent `math domain` errors.