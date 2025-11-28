# Wind & Solar Hybrid Simulation — VE + MPPT + Daily Energy Scan

This project is an **interactive 3D simulation** of a windmill and solar PV field with:

- A rotating **wind turbine**
- **Single-axis tracking** solar strings
- **Physics-based I–V curves** for the PV string
- **Current-controlled MPPT inverter**
- **Voltage Equalization (VE) optimizer** per group of panels
- **Live energy scan** and **whole-day energy simulation** (sunrise → sunset)

The entire app is built in the browser using **HTML, CSS, and JavaScript** with **Three.js**, **SunCalc**, and **Chart.js**.

---

## Scene & PV Layout

### Solar Panels & Strings

- The solar field consists of **multiple single-axis tracking strings**.
- Each **target string**:
  - Has **28 panels in series**.
  - Each panel is modeled internally as **3 substrings**, so the full string has **84 substrings**.
  - Uses **substring-level irradiance** to model **partial shading** from the windmill shadow.
- A **highlighted string** (closest to the windmill) is used as the **active analysis string** for:
  - I–V curve generation
  - MPPT operation
  - VE optimizer behavior
  - Energy calculations

### Windmill

- A **3-blade horizontal-axis wind turbine** is modeled with:
  - **Tower**, **nacelle**, **hub**, and **blades**.
  - **Blade geometry** with varying chord and twist from root to tip.
- The turbine produces **shadow geometry** which is projected onto the ground and solar strings.
- Shadow length can be **measured dynamically** with the **“Analyze Shadow”** tool.

---

##  Electrical Modeling

### Physics-Based PV String Model

- Uses a **Shockley diode-based model** to generate:
  - **Advanced I–V curves** for:
    - **Unshaded (ideal)** string
    - **Shaded** string with substring-level shading
- Each substring has:
  - Photocurrent (**IL**),
  - Diode saturation current (**I₀**),
  - Thermal voltage (**Vt**),
  - Series resistance (**Rs**),
  - Shunt resistance (**Rsh**),
  - **Bypass diode behavior**.
- **Cell temperature** is estimated from irradiance → affects **Voc**, **Isc**, and the full I–V shape.

---

##  MPPT & Inverter

The simulation includes a **current-controlled inverter** with configurable **MPPT algorithms**.

### MPPT Algorithms

You can select MPPT mode in the **MPPT Settings** panel:

- **Perturb & Observe (P&O)**  
  - Adjusts **current setpoint** (Iset) based on power comparison:
    - If new power < old power → reverse direction.
    - If new power > old power → continue in same direction.
  - Parameters:
    - **P&O Step (A)**: size of current step.
    - **MPPT Frequency (Hz)**: how often the algorithm updates.

- **Incremental Conductance (IncCond)**  
  - Uses the condition **dI/dV = -I/V** at MPP.
  - Adjusts current based on **error between dI/dV and -I/V**.
  - Parameters:
    - **IncCond Step (A)**
    - **MPPT Frequency (Hz)**

### Inverter Model

- Modeled as a **DC/AC interface** with:
  - **Efficiency (η)** (configurable, default ~0.97)
  - Computes:
    - **P_DC = Vop × Iop**
    - **P_AC = η × P_DC**
- UI shows:
  - MPPT operating point **(Vop, Iop)**
  - **Pmax (shaded)** vs **Pmax (ideal)**  
  - Instantaneous **P_DC** and **P_AC**

You can:

- **Pause / resume MPPT**
- **Reset MPPT** to re-initialize from MPP.

---

##  Voltage Equalization (VE) Optimizer

The system includes a **Voltage Equalization (VE) optimizer** that operates on **groups of panels** to reduce mismatch under partial shading.

### Grouping

- Panels are grouped by **DC optimizers**:
  - **4 panels per optimizer**
  - For 28 panels → **7 optimizers** per string.

### VE Behavior

For each optimizer group, VE adjusts a **duty cycle (0–1)** that scales the **panel current**:

- **Duty down** (lower current) for **heavily mismatched / shaded** groups.
- **Duty up** (towards 1.0) when group voltage recovers or is near the global group max.
- Tracks:
  - **Group average voltage**
  - **Sudden rises / surges**
  - **Shading state** over time.

### VE Parameters (Configurable in UI)

- VE On/Off
- **Duty Cycle Duration (ms)** (reset window)
- **Step Interval (ms)** (timing for VE logic)
- **Duty Step (%)**
- **Minimum Duty (%)**
- **Surge Threshold (V)**
- Panel indices to **trace** (e.g., `2,7,11,20`)

A dedicated **VE chart** plots **per-panel voltages over time** like an **ECG trace**, for selected panel indices.

---

##  Panel Voltage & I–V Visualization

### Panel Voltage Panel

- Shows **per-panel voltage** (P01–P28) at the current operating point.
- Displays:
  - MPPT operating voltage and current
  - Sum of per-panel voltages (ΣV)
- Updated live as **VE and MPPT** adjust the operating point.

### I–V & P–V Chart

Includes:

- **Shaded I–V curve**
- **Unshaded (ideal) I–V curve**
- **P–V curve for shaded condition**
- **MPPT operating point** marker
- Numerical readout:
  - Pmax (shaded) vs Pmax (ideal)
  - Power loss %
  - Vop, Iop, P_DC, P_AC

---

##  Energy Simulation Features

### 1. Live Energy Scan (0.01 s Time Resolution)

- **Energy Scan (0.01 s)** uses a **high-resolution time step** (0.01 s) to:
  - March through the day between sunrise and sunset.
  - Accumulate:
    - **Ideal (unshaded) energy**
    - **Max (shaded) energy**
    - **Actual DC energy (with MPPT)**
    - **Actual AC energy (with inverter efficiency)**
- Controls:
  - **Start Live Scan** / **Pause**
  - **Reset**
- Live energy is displayed in a dedicated **Energy panel** in kWh.

### 2. Whole-Day Energy (Sunrise → Sunset)

- **Whole-Day Energy** computes kWh over the entire day, using:
  - Adaptive time-stepping:
    - **Fine steps** when shading or power changes quickly.
    - **Larger steps** when conditions are stable.
  - VE-aware and MPPT-driven simulation.
- Outputs:
  - **Ideal (unshaded) energy (kWh)**
  - **Max (shaded) energy (kWh)**
  - **Actual DC energy (kWh)**
  - **Actual AC energy (kWh)**
- A **“Whole-Day Energy (kWh)”** panel displays these values along with status text during calculation.

---

##  UI Overview

- **3D controls:**
  - Drag → rotate camera
  - Scroll → zoom
- **Top-left**:
  - Info panel (features summary)
- **Top-right**:
  - Date picker
  - Time-of-day slider
  - Buttons for:
    - Shadow analysis
    - Solar string analysis (I–V and MPPT)
    - Panel voltages
    - MPPT settings
    - VE settings
    - Live energy scan
    - Whole-day energy
- **Left side:**
  - Play/Pause animation
  - Speed control (0.01x–2x)
  - Compass indicator (N/E/S/W)
- **Bottom center:**
  - Time display
  - Measurement display (shadow length)
- **Bottom panels:**
  - I–V chart
  - Energy panel
  - Daily energy summary
  - VE & Panel voltage panels (toggleable)

---

##  Tech Stack

- **Three.js** — 3D graphics and scene rendering
- **SunCalc** — solar position and sun times (sunrise, sunset)
- **Chart.js** — I–V curves, panel voltage bars, VE traces
- **HTML5 / CSS3 / JavaScript (ES Modules)**

---

##  Running the Simulation

1. Save the HTML file as `index.html`.
2. Serve it using any static server:
   - VS Code Live Server
   - `python -m http.server`
   - Or any simple HTTP server.
3. Open in a modern browser (Chrome, Edge, etc.)

---

##  Notes

- All PV, MPPT, VE, and energy calculations run **fully in the browser** (no backend).
- The model is designed to make **shading, MPPT tracking, and optimizer behavior visually and numerically intuitive** for teaching, debugging, or research demos.
