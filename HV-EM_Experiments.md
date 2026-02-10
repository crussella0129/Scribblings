# The Open Source HPM Turret: A Research-Grade Framework for Directed Microwave Countermeasures

**Author:** Charles (Thread & Signal)
**Date:** February 2026
**Revision:** 1.0 — DRAFT
**Classification:** Open Source / UNCLASSIFIED
**License:** CC BY-NC-SA 4.0

---

> **DISCLAIMER:** This document is an engineering research paper and educational reference. It describes systems that, at certain power levels, become regulated or prohibited devices under federal and international law. The author and contributors assume no liability for misuse. **Read Section 9 (Legal Framework) and Section 10 (Safety) in their entirety before sourcing any components.** Failure to do so may result in federal prosecution, serious injury, or death.

---

## Table of Contents

1. [Abstract](#1-abstract)
2. [Introduction & Motivation](#2-introduction--motivation)
3. [Threat Model & Physical Basis](#3-threat-model--physical-basis)
4. [System Architecture](#4-system-architecture)
5. [Subsystem Design](#5-subsystem-design)
   - 5.1 Detection Array
   - 5.2 Tracking & Gimbal
   - 5.3 Controller (FPGA/Jetson)
   - 5.4 Emitter — Smart Jammer (GaN Solid-State)
   - 5.5 Emitter — High-Power Magnetron (Informational)
   - 5.6 Power Supply & Energy Storage
   - 5.7 Cooling
6. [Bill of Materials](#6-bill-of-materials)
7. [Build Instructions](#7-build-instructions)
8. [Testing & Calibration](#8-testing--calibration)
9. [Legal Framework & Controlled Device Thresholds](#9-legal-framework--controlled-device-thresholds)
10. [Safety — How Not to Die](#10-safety--how-not-to-die)
11. [Future Work](#11-future-work)
12. [References](#12-references)
13. [Appendices](#13-appendices)

---

## 1. Abstract

This paper presents a modular, open-source framework for a directed microwave countermeasure turret system — a platform capable of detecting, tracking, and disrupting directed radiofrequency (RF) threats in the 1–18 GHz band. The system is designed in a tiered architecture: Tier 1 (fully legal, low-power detection and tracking), Tier 2 (low-power jamming requiring FCC experimental licensing), and Tier 3 (high-power microwave emission requiring federal authorization and representing a controlled device). Complete bills of materials, build procedures, and safety protocols are provided for Tiers 1 and 2. Tier 3 is documented at a design level for academic completeness but includes explicit legal and safety gates that must be satisfied before implementation.

The platform is built around the NVIDIA Jetson Orin Nano Super for real-time signal processing and gimbal control, commodity software-defined radios for detection, and a servo-driven gimbal for beam pointing. The emitter subsystem is treated as a hot-swappable module, allowing the platform to serve as a general-purpose directed energy research testbed.

---

## 2. Introduction & Motivation

### 2.1 The Havana Syndrome Hypothesis

Beginning in 2016, U.S. and Canadian diplomatic personnel in Havana, Cuba reported a cluster of neurological symptoms — persistent headaches, vertigo, cognitive dysfunction, and auditory phenomena — consistent with exposure to directed microwave energy. Subsequent incidents were reported at diplomatic postings worldwide.

The leading technical hypothesis, supported by a 2020 National Academies of Sciences report, attributes these symptoms to **pulsed radiofrequency energy** in the low-GHz band, exploiting the **Frey auditory effect** — a phenomenon where pulsed microwaves induce thermoelastic pressure waves in cranial tissue, perceived as clicking, buzzing, or pain.

Whether or not this specific attribution is correct, the underlying physics is unambiguous: directed pulsed microwave energy at sufficient power density causes measurable biological effects. The absence of widely available countermeasure technology represents a gap in civilian defensive capability.

### 2.2 Design Philosophy

This project follows three principles:

1. **Defense-first design.** The primary mission is detection, characterization, and disruption of incoming RF threats — not offensive emission.
2. **Tiered legality.** The system is explicitly partitioned into legal tiers so that builders can stop at any point and still have a functional, lawful system.
3. **Modular architecture.** The gimbal, controller, and detection subsystems are payload-agnostic. The emitter is a swappable module.

### 2.3 Who This Is For

- RF engineers and amateur radio operators studying directed energy phenomena
- Security researchers evaluating RF vulnerability of facilities
- Academic groups studying electronic countermeasures
- Makers and robotics enthusiasts building advanced gimbal/tracking platforms

### 2.4 Who This Is NOT For

- Anyone seeking to build an offensive weapon. The physics documented here are well-known; the contribution of this paper is in *defensive* integration.
- Anyone unwilling to read and comply with Section 9 (Legal) and Section 10 (Safety).

---

## 3. Threat Model & Physical Basis

### 3.1 The Frey Auditory Effect

When pulsed microwave radiation is absorbed by cranial tissue, rapid (microsecond-scale) thermoelastic expansion of water in tissue generates acoustic pressure waves detectable by the cochlea. The threshold for auditory perception is:

```
Specific Absorption Rate (SAR) for Frey Effect:
  SAR_threshold ≈ 1.6 W/kg (peak, per pulse)

Where:
  SAR = (conductivity × E_field²) / (2 × tissue_density)
  
  conductivity ≈ 1.5 S/m  (brain tissue at 3 GHz)
  tissue_density ≈ 1040 kg/m³
```

For harmful effects (sustained headache, cognitive disruption, tissue damage), the required SAR rises to **4–10 W/kg sustained** or significantly higher peak values.

### 3.2 Threat Signal Characteristics

Based on open-source analysis of the Havana syndrome hypothesis:

| Parameter | Estimated Range |
|-----------|----------------|
| Frequency | 1–10 GHz (likely 2–4 GHz) |
| Modulation | Pulsed, 100 Hz – 10 kHz PRF |
| Pulse width | 1–100 μs |
| Peak power at source | 1 kW – 100 kW |
| Effective range | 5–50 m (through walls: reduced) |
| Beam width | 5°–20° (depending on antenna) |

### 3.3 Free-Space Path Loss

The power density at a target at distance `r` from an emitter with gain `G` and transmit power `P_t`:

```
Power Density at Target:
  S = (P_t × G) / (4 × pi × r²)    [W/m²]

Example:
  P_t = 10 kW, G = 100 (20 dBi dish), r = 30 m
  S = (10000 × 100) / (4 × 3.14159 × 900)
  S ≈ 88.4 W/m²

For reference:
  FCC general public exposure limit at 3 GHz = 10 W/m²
  Occupational limit = 50 W/m²
  Frey effect threshold ≈ 5–50 W/m² (pulsed, frequency-dependent)
```

### 3.4 Wall Attenuation

Common building materials provide limited but non-trivial attenuation:

| Material | Approximate Attenuation at 3 GHz |
|----------|----------------------------------|
| Drywall (12 mm) | 1–2 dB |
| Concrete (200 mm) | 10–15 dB |
| Brick (100 mm) | 5–10 dB |
| Glass (6 mm) | 0.5–1 dB |
| Reinforced concrete | 15–25 dB |

Glass windows are essentially transparent. This is the primary vulnerability in any facility.

---

## 4. System Architecture

### 4.1 High-Level Block Diagram

```
┌─────────────────────────────────────────────────────────┐
│                    TURRET PLATFORM                       │
│                                                         │
│  ┌──────────────┐    ┌──────────────┐    ┌───────────┐ │
│  │  Detection    │───▶│  Controller  │───▶│  Gimbal   │ │
│  │  Array        │    │  (Jetson)    │    │  (Az/El)  │ │
│  │  (SDR × 3+)  │    │              │    │           │ │
│  └──────────────┘    │  - GNU Radio │    └─────┬─────┘ │
│                      │  - Tracking  │          │       │
│                      │  - ML Classif│    ┌─────┴─────┐ │
│                      │  - Gimbal Ctl│    │  Emitter  │ │
│                      └──────────────┘    │  Module   │ │
│                                          │  (swap.)  │ │
│                                          └───────────┘ │
│                                                         │
│  ┌──────────────┐    ┌──────────────┐                   │
│  │  Power Supply │    │  Cooling     │                   │
│  │  (Mains/Gen) │    │  System      │                   │
│  └──────────────┘    └──────────────┘                   │
└─────────────────────────────────────────────────────────┘
```

### 4.2 Tier Definitions

**Tier 1 — DETECT & TRACK (Fully Legal)**
- SDR-based RF spectrum monitoring
- ML-based anomaly detection
- Servo gimbal with camera tracking
- Passive only — no RF emission
- No license required (receive-only)

**Tier 2 — LOW-POWER JAMMER (FCC Experimental License Required)**
- Adds a low-power (<1W) GaN amplifier and horn antenna
- Capable of localized interference within a shielded test environment
- Requires FCC Part 5 Experimental Radio Service license
- Useful for testing and validating the detection/response loop

**Tier 3 — HIGH-POWER COUNTERMEASURE (Federal Authorization Required)**
- Adds high-power magnetron or high-power GaN amplifier array
- Marx generator or capacitor bank energy storage
- Constitutes a directed energy device under multiple federal regulations
- Requires DOD/DOE coordination, ITAR considerations, or law enforcement authorization
- **DO NOT BUILD without legal counsel and federal authorization**

---

## 5. Subsystem Design

### 5.1 Detection Array

#### 5.1.1 Concept

Three or more software-defined radios arranged in a known geometric pattern enable both spectrum monitoring and direction-of-arrival (DOA) estimation via time-difference-of-arrival (TDOA) or phase interferometry.

#### 5.1.2 SDR Selection

| SDR | Frequency Range | Bandwidth | Cost | Notes |
|-----|----------------|-----------|------|-------|
| RTL-SDR v4 | 24–1766 MHz | 2.4 MHz | ~$30 | Requires downconverter for >1.7 GHz |
| HackRF One | 1–6 GHz | 20 MHz | ~$350 | Covers primary threat band directly |
| Ettus USRP B210 | 70 MHz–6 GHz | 56 MHz | ~$2,100 | Research-grade, best bandwidth |
| LimeSDR Mini 2.0 | 10 MHz–3.5 GHz | 30.72 MHz | ~$250 | Good compromise of cost/performance |

**Recommended for Tier 1:** 3× HackRF One or 3× LimeSDR Mini 2.0, arranged in an equilateral triangle with 0.5 m spacing (approximately λ/2 at 3 GHz for optimal phase interferometry).

#### 5.1.3 Antenna Array for Detection

Each SDR feeds a **wideband log-periodic dipole antenna (LPDA)** or **wideband horn** covering 1–6 GHz. Commercial LPDAs in this range are available from Aaronia, Rohde & Schwarz, or can be fabricated from PCB.

#### 5.1.4 Direction of Arrival — The Math

For a two-element interferometer with spacing `d`, the angle of arrival `θ` relative to the baseline:

```
Phase Difference Method:
  θ = arcsin(Δφ × λ / (2 × pi × d))

Where:
  Δφ = phase difference between signals at two antennas (radians)
  λ  = wavelength of the detected signal
  d  = antenna spacing

For TDOA with three antennas:
  Δt_12 = time difference of arrival between antenna 1 and 2
  Δt_13 = time difference of arrival between antenna 1 and 3
  
  Bearing is solved by intersection of two hyperbolas defined by:
    |r1 - r2| = c × Δt_12
    |r1 - r3| = c × Δt_13
  
  Where c = speed of light ≈ 3 × 10⁸ m/s
```

At 3 GHz with 20 MHz SDR bandwidth, timing resolution is approximately:

```
  timing_resolution ≈ 1 / bandwidth = 1 / 20e6 = 50 ns
  distance_resolution = c × timing_resolution = 3e8 × 50e-9 = 15 m
```

This is coarse for TDOA. Phase interferometry provides much better angular resolution (sub-degree) but requires coherent SDRs (shared clock reference). The **LimeSDR Mini** supports external clock input, making coherent operation feasible.

#### 5.1.5 Signal Processing Pipeline

```python
"""
Detection pipeline running on Jetson Orin via GNU Radio + Python
"""
import numpy as np
from scipy.signal import welch, find_peaks

class ThreatDetector:
    def __init__(self, sdr_array, baseline_duration_s=60):
        self.sdr_array = sdr_array        # List of SDR interfaces
        self.baseline = None
        self.baseline_duration = baseline_duration_s
    
    def calibrate_baseline(self):
        """Record ambient RF environment as baseline."""
        spectra = []
        for _ in range(self.baseline_duration):
            for sdr in self.sdr_array:
                samples = sdr.read_samples(256 * 1024)
                freqs, psd = welch(samples, fs=sdr.sample_rate, nperseg=1024)
                spectra.append(psd)
        self.baseline = np.mean(spectra, axis=0)
        self.baseline_std = np.std(spectra, axis=0)
    
    def detect_anomalies(self, threshold_sigma=6):
        """
        Compare current spectrum to baseline.
        Flag bins exceeding threshold_sigma standard deviations.
        """
        for sdr in self.sdr_array:
            samples = sdr.read_samples(256 * 1024)
            freqs, psd = welch(samples, fs=sdr.sample_rate, nperseg=1024)
            deviation = (psd - self.baseline) / self.baseline_std
            anomaly_bins = np.where(deviation > threshold_sigma)[0]
            if len(anomaly_bins) > 0:
                return self._characterize_threat(freqs, psd, anomaly_bins)
        return None
    
    def _characterize_threat(self, freqs, psd, anomaly_bins):
        """Extract threat signal parameters."""
        peak_bin = anomaly_bins[np.argmax(psd[anomaly_bins])]
        return {
            'frequency_hz': freqs[peak_bin],
            'power_db': 10 * np.log10(psd[peak_bin]),
            'bandwidth_hz': len(anomaly_bins) * (freqs[1] - freqs[0]),
            'timestamp': time.time()
        }
    
    def estimate_bearing(self, threat):
        """
        Phase interferometry across SDR array.
        Requires coherent (clock-shared) SDRs.
        """
        phases = []
        for sdr in self.sdr_array:
            samples = sdr.read_samples(256 * 1024)
            # Bandpass filter around threat frequency
            filtered = self._bandpass(samples, threat['frequency_hz'], 
                                       sdr.sample_rate)
            phase = np.angle(np.mean(filtered))
            phases.append(phase)
        
        # Solve DOA from phase differences
        # (simplified for 2D, linear array)
        wavelength = 3e8 / threat['frequency_hz']
        d = self.sdr_array.spacing_m
        delta_phi = phases[1] - phases[0]
        bearing_rad = np.arcsin(delta_phi * wavelength / (2 * np.pi * d))
        return np.degrees(bearing_rad)
```

### 5.2 Tracking & Gimbal

#### 5.2.1 Mechanical Design

The gimbal requires two axes of rotation:

- **Azimuth** (horizontal): 0–360° continuous or ±180°
- **Elevation** (vertical): -10° to +90°

The gimbal must support the mass of the emitter module (estimated 5–15 kg depending on tier) with minimal backlash for accurate pointing.

#### 5.2.2 Motor Selection

| Motor Type | Torque | Precision | Cost | Notes |
|-----------|--------|-----------|------|-------|
| NEMA 23 Stepper | 1.2–3.0 N·m | 1.8°/step (0.05° with 32× microstepping) | $20–50 | Simple, good precision, but resonance at mid-speeds |
| NEMA 34 Stepper | 4.0–12.0 N·m | Same | $50–120 | For heavier payloads |
| Brushless DC (BLDC) with encoder | Varies | Encoder-dependent (0.01° achievable) | $100–400 | Smooth, fast, closed-loop |
| Servo actuator (Dynamixel MX-106) | 8.4 N·m | 0.088° | ~$450 each | Integrated controller, easiest to implement |

**Recommended:** Dynamixel MX-106 for rapid prototyping, or NEMA 23 steppers with TMC2209 silent drivers and AS5600 magnetic encoders for a cost-optimized build.

#### 5.2.3 Gimbal Structure

The gimbal frame should be fabricated from **6061-T6 aluminum** for strength-to-weight ratio (yield strength ~276 MPa, density 2,700 kg/m³) or **3D printed in PETG/ABS** for prototyping with metal reinforcement at bearing points.

Key mechanical parameters:

```
Required Torque Calculation:
  torque = moment_of_inertia × angular_acceleration + friction_torque

  For a 10 kg payload at 0.3 m radius:
    I = m × r² = 10 × 0.09 = 0.9 kg·m²
  
  For 180°/s² acceleration (reaching 90°/s in 0.5 s):
    angular_accel = pi rad/s²
    torque_dynamic = 0.9 × pi ≈ 2.83 N·m
    
  Add ~30% for friction and safety:
    torque_required ≈ 3.7 N·m

  → NEMA 23 with 2:1 gear reduction, or Dynamixel MX-106 direct-drive
```

#### 5.2.4 Control Loop

The gimbal controller runs a cascaded PID loop: outer loop on position, inner loop on velocity, updating at ≥100 Hz.

```python
"""
Gimbal controller for Jetson Orin GPIO / UART / CAN interface
"""
import time

class PIDController:
    def __init__(self, kp, ki, kd, output_limits=(-1.0, 1.0)):
        self.kp = kp
        self.ki = ki
        self.kd = kd
        self.limits = output_limits
        self.integral = 0.0
        self.prev_error = 0.0
        self.prev_time = time.time()
    
    def update(self, setpoint, measurement):
        now = time.time()
        dt = now - self.prev_time
        if dt <= 0:
            return 0.0
        
        error = setpoint - measurement
        self.integral += error * dt
        derivative = (error - self.prev_error) / dt
        
        output = (self.kp * error + 
                  self.ki * self.integral + 
                  self.kd * derivative)
        
        output = max(self.limits[0], min(self.limits[1], output))
        
        self.prev_error = error
        self.prev_time = now
        return output


class GimbalController:
    def __init__(self, az_motor, el_motor):
        self.az_pid = PIDController(kp=2.0, ki=0.1, kd=0.5)
        self.el_pid = PIDController(kp=2.0, ki=0.1, kd=0.5)
        self.az_motor = az_motor
        self.el_motor = el_motor
    
    def point_to(self, target_az_deg, target_el_deg):
        """Command gimbal to point at target bearing."""
        current_az = self.az_motor.read_position()
        current_el = self.el_motor.read_position()
        
        az_cmd = self.az_pid.update(target_az_deg, current_az)
        el_cmd = self.el_pid.update(target_el_deg, current_el)
        
        self.az_motor.set_velocity(az_cmd)
        self.el_motor.set_velocity(el_cmd)
    
    def is_on_target(self, target_az, target_el, tolerance_deg=1.0):
        """Check if gimbal is pointed within tolerance."""
        az_err = abs(target_az - self.az_motor.read_position())
        el_err = abs(target_el - self.el_motor.read_position())
        return az_err < tolerance_deg and el_err < tolerance_deg
```

### 5.3 Controller — NVIDIA Jetson Orin Nano Super

#### 5.3.1 Why Jetson

The Jetson Orin Nano Super provides:

- 67 TOPS AI inference performance for ML-based signal classification
- 8 GB LPDDR5 for buffering SDR data streams
- USB 3.2 for high-bandwidth SDR interfaces
- 40-pin GPIO header for motor control
- CUDA acceleration for real-time FFT processing
- Ubuntu/JetPack ecosystem for GNU Radio compatibility

#### 5.3.2 Software Stack

```
┌─────────────────────────────┐
│  Application Layer          │
│  - Threat Classifier (PyTorch/TensorRT)
│  - Gimbal Controller        │
│  - Engagement Logic         │
│  - Web Dashboard (Flask)    │
├─────────────────────────────┤
│  Signal Processing Layer    │
│  - GNU Radio                │
│  - CuPy/NumPy for FFT      │
│  - SciPy for filtering      │
├─────────────────────────────┤
│  Hardware Abstraction       │
│  - SoapySDR (SDR interface) │
│  - Dynamixel SDK / GPIO     │
│  - UART/CAN for motors      │
├─────────────────────────────┤
│  OS: JetPack 6.x / Ubuntu 22.04
│  + GNU Radio 3.10+          │
│  + CUDA 12.x                │
└─────────────────────────────┘
```

#### 5.3.3 Real-Time Constraints

The detect-to-track latency budget:

```
Signal acquisition:     ~10 ms  (SDR buffer)
FFT + anomaly detect:   ~5 ms   (CUDA-accelerated)
DOA estimation:         ~2 ms
Gimbal command:         ~1 ms
Gimbal slew to target:  ~500 ms (worst case, 180° at 360°/s)
─────────────────────────────────
Total:                  ~520 ms worst case
```

This is sufficient for a threat that must maintain beam-on-target for seconds to minutes to cause harm. Sub-second response effectively breaks the engagement.

### 5.4 Emitter — Tier 2: Smart Jammer (GaN Solid-State)

#### 5.4.1 Architecture

```
DDS Waveform Generator → GaN Power Amplifier → Waveguide → Horn Antenna
        ↑
FPGA/Jetson (threat frequency + modulation data)
```

#### 5.4.2 Key Components

**Direct Digital Synthesizer (DDS):** An AD9914 or AD9959 evaluation board provides frequency-agile waveform generation up to 1.4 GHz (with mixing/upconversion for higher bands), with sub-Hz frequency resolution and nanosecond switching.

**GaN Power Amplifier:** For Tier 2 (sub-1W for licensed experimental use), a commercial evaluation board like the Qorvo TGA2594 (2–6 GHz, 4W saturated) operated well below saturation. For Tier 3 concepts, GaN MMIC amplifiers from Wolfspeed/Qorvo can reach 50–200 W per device in the S-band.

**Horn Antenna:** A standard-gain pyramidal horn for the target frequency band. At 3 GHz, dimensions approximately:

```
Horn Antenna Sizing (3 GHz, ~20 dBi gain):
  Aperture width:   a = 3.3 × λ = 3.3 × 0.1 m = 0.33 m
  Aperture height:  b = 2.5 × λ = 2.5 × 0.1 m = 0.25 m  
  Horn length:      L ≈ 0.4 m
  Waveguide feed:   WR-284 (72.14 × 34.04 mm)

  Can be fabricated from 0.5 mm copper sheet, brass, or aluminum.
  Joints soldered (copper/brass) or welded (aluminum).
```

#### 5.4.3 Jammer Waveform Strategy

Rather than attempting coherent phase cancellation (which requires sub-nanosecond timing), the jammer employs **noise jamming** — broadcasting wideband noise centered on the detected threat frequency:

```python
def generate_jam_waveform(threat_freq_hz, bandwidth_hz, sample_rate):
    """
    Generate band-limited noise centered on threat frequency.
    
    Disrupts coherent pulse structure required for Frey effect
    without needing phase synchronization.
    """
    n_samples = int(sample_rate * 0.001)  # 1 ms block
    
    # Complex Gaussian noise
    noise = (np.random.randn(n_samples) + 
             1j * np.random.randn(n_samples))
    
    # Bandpass filter to threat band
    # (implemented as frequency-domain windowing)
    spectrum = np.fft.fft(noise)
    freqs = np.fft.fftfreq(n_samples, 1/sample_rate)
    
    mask = np.abs(freqs - threat_freq_hz) < (bandwidth_hz / 2)
    spectrum *= mask
    
    return np.fft.ifft(spectrum)
```

### 5.5 Emitter — Tier 3: High-Power Magnetron (INFORMATIONAL ONLY)

> **⚠️ STOP. This subsection is provided for academic completeness only. Building this subsystem without federal authorization constitutes a violation of FCC regulations, potentially ITAR, and creates immediate risk of lethal arc flash, X-ray exposure, and RF burns. See Sections 9 and 10.**

#### 5.5.1 Architecture

```
Mains AC → HV Transformer → Rectifier → Marx Generator → PFN → Magnetron → Waveguide → Horn/Dish
```

#### 5.5.2 Marx Generator Design Principles

A Marx generator charges `n` capacitors in parallel through resistors, then discharges them in series through spark gaps, multiplying voltage by `n`:

```
Marx Generator Output:
  V_out = n × V_charge
  E_stored = n × (0.5 × C × V_charge²)
  
  Example: 10 stages, 3 kV charge, 10 nF per stage
    V_out = 30 kV
    E_stored = 10 × 0.5 × 10e-9 × (3000)² = 0.45 J per pulse
    
  For meaningful HPM output, E_stored should be 10–1000 J per pulse:
    At 3 kV/stage, 10 stages: C = 220 nF – 22 μF per stage
    (Physically: large pulse-rated capacitors, ~$50–200 each)
```

**Critical safety note on Marx generators:** At 30 kV and above, the arc flash energy in a fault condition is:

```
Arc Flash Energy:
  E_arc = 0.5 × C_total × V²
  
  For C_total = 2.2 μF at 30 kV:
    E_arc = 0.5 × 2.2e-6 × (30000)² = 990 J
    
  For comparison:
    - 1 J across skin = painful burn
    - 10 J across chest = potentially lethal cardiac disruption
    - 100 J = severe burn, likely lethal electrical injury
    - 990 J = VAPORIZATION of copper conductors, intense UV flash,
              shrapnel from exploding components, lethal at any contact
```

**This is not a metaphor. "Vaporizing arc flash" means the copper wire literally transitions to a plasma state, creating an explosion.** At these energy levels, PPE alone is insufficient — the system must be designed with intrinsic safety (dump resistors, safety interlocks, dead-man switches, physical barriers).

#### 5.5.3 Pulse Forming Network (PFN)

The PFN shapes the Marx generator's sharp discharge into a flat-top pulse suitable for driving the magnetron:

```
PFN Design (Type E Guillemin network):
  For a pulse of width τ and impedance Z:
    
    L_section = Z × τ / (2 × n_sections)
    C_section = τ / (2 × n_sections × Z)
    
  Example: τ = 1 μs, Z = 400 Ω, n_sections = 5
    L = 400 × 1e-6 / 10 = 40 μH per section
    C = 1e-6 / (10 × 400) = 250 pF per section
```

#### 5.5.4 Magnetron

Cavity magnetrons in the 2–4 GHz range are commercially available as industrial heating components (ISM band at 2.45 GHz). A typical CW magnetron from a commercial microwave oven produces ~1 kW at 2.45 GHz. Industrial units (e.g., from Richardson Electronics, CPI, or Toshiba) produce 5–100 kW pulsed.

**Note:** Acquiring surplus military magnetrons may trigger ITAR or EAR (Export Administration Regulations) scrutiny. Commercial ISM-band magnetrons are unrestricted.

### 5.6 Power Supply & Energy Storage

#### Tier 1 (Detection Only)
- Jetson Orin: 15W nominal, 25W peak
- SDRs: ~2.5W each × 3 = 7.5W
- Gimbal motors: 20–50W peak
- **Total: ~80W peak → standard 12V/10A power supply**

#### Tier 2 (Low-Power Jammer)
- Add GaN amplifier: 5–20W DC input
- DDS board: 5W
- **Total: ~100W peak → 12V/15A supply or 24V/5A**

#### Tier 3 (High-Power — Informational)
- Marx generator charging: depends on PRF and energy per pulse
- For 100 J at 10 Hz: 1 kW average charging power
- HV transformer + rectifier from mains (240V recommended for >500W)
- Cooling system: 200–500W
- **Total: 1.5–5 kW → dedicated 240V circuit or generator**

### 5.7 Cooling

#### Tier 1–2
Passive heatsinking on the Jetson and amplifier is likely sufficient. Add a 120mm fan if operating in enclosed space.

#### Tier 3
The magnetron dissipates approximately 50% of input power as heat. For a 10 kW peak, 10 Hz, 1 μs pulse magnetron:

```
Average thermal dissipation:
  P_heat = P_peak × duty_cycle × (1 - efficiency)
  P_heat = 10000 × (1e-6 × 10) × 0.5
  P_heat = 0.05 W average (for this low duty cycle)
  
  But: the Marx generator and PFN also dissipate heat,
  and charging circuitry runs continuously.
  Budget 200–500W for thermal management.
```

A closed-loop liquid cooling system (automotive intercooler + pump + reservoir) is appropriate. Route coolant through a copper block bolted to the magnetron anode.

---

## 6. Bill of Materials

### 6.1 Tier 1 — Detect & Track (Estimated: $1,800 – $3,500)

| Item | Qty | Unit Cost | Total | Source |
|------|-----|-----------|-------|--------|
| NVIDIA Jetson Orin Nano Super Dev Kit | 1 | $250 | $250 | NVIDIA / Arrow / Seeed |
| HackRF One | 3 | $350 | $1,050 | Great Scott Gadgets |
| — *OR* LimeSDR Mini 2.0 | 3 | $250 | $750 | Lime Microsystems |
| Wideband LPDA Antenna (1–6 GHz) | 3 | $80–200 | $240–600 | Aaronia / AliExpress |
| Dynamixel MX-106 Servo | 2 | $450 | $900 | Robotis |
| — *OR* NEMA 23 Stepper + TMC2209 + AS5600 | 2 sets | $60 | $120 | StepperOnline / Amazon |
| Gimbal frame (aluminum or 3D printed) | 1 | $50–200 | $50–200 | Custom / SendCutSend |
| SMA cables, adapters, connectors | lot | — | $100 | Digikey / Mouser |
| 12V 10A power supply | 1 | $30 | $30 | Amazon / Mouser |
| Ethernet switch (for SDR data) | 1 | $30 | $30 | Amazon |
| Misc (wiring, mounting, enclosure) | lot | — | $150 | Various |
| **Subtotal (HackRF path)** | | | **~$2,800** | |
| **Subtotal (LimeSDR path)** | | | **~$2,200** | |

### 6.2 Tier 2 — Low-Power Jammer (Additional: $500 – $1,500)

| Item | Qty | Unit Cost | Total | Source |
|------|-----|-----------|-------|--------|
| AD9959 DDS Evaluation Board | 1 | $150–300 | $150–300 | Analog Devices / eBay |
| GaN PA Eval Board (e.g., Qorvo TGA2594-SM) | 1 | $200–500 | $200–500 | Qorvo / Richardson RFPD |
| WR-284 Waveguide section (0.3m) | 1 | $50–150 | $50–150 | Pasternack / eBay |
| Pyramidal horn antenna (2–4 GHz) | 1 | $100–300 | $100–300 | Pasternack / Custom fab |
| RF shielded test enclosure (for legal testing) | 1 | $200–500 | $200–500 | Ramsey / Amazon |
| SMA-to-waveguide adapter | 1 | $30 | $30 | Pasternack |
| **Subtotal** | | | **~$750–1,500** | |

### 6.3 Tier 3 — High-Power (Additional: $2,000 – $10,000+)

> **⚠️ DO NOT PURCHASE THESE COMPONENTS without first completing the legal authorization process described in Section 9.** Possession of an assembled high-power directed RF system without authorization may itself constitute a violation.

| Item | Qty | Unit Cost | Total | Notes |
|------|-----|-----------|-------|-------|
| HV pulse capacitors (10 nF–10 μF, 3–5 kV) | 10–20 | $20–200 | $200–2,000 | Must be pulse-rated (not electrolytic) |
| Spark gap assemblies or thyratron | 10 | $50–500 | $500–5,000 | Thyratrons are surplus; spark gaps can be fabricated |
| HV charging resistors (wirewound) | 20 | $5 | $100 | Must be rated for pulse duty |
| HV transformer (MOT or custom) | 1 | $50–300 | $50–300 | Microwave oven transformers are cheap but dangerous |
| HV rectifier diodes (strings) | 1 set | $50 | $50 | CL01-12 or equivalent |
| Magnetron (CW or pulsed, 2.45 GHz ISM) | 1 | $100–2,000 | $100–2,000 | Surplus or commercial ISM |
| PFN inductors (custom wound) | 5–10 | $10–50 | $50–500 | Air-core, must handle pulse current |
| Liquid cooling system | 1 | $100–300 | $100–300 | Automotive intercooler + pump |
| HV insulation (silicone potting, HDPE) | lot | — | $200 | McMaster-Carr |
| Safety interlocks, dump resistors, dead-man | lot | — | $200 | Custom |
| **Subtotal** | | | **~$2,000–10,000** | |

---

## 7. Build Instructions

### 7.1 Tier 1 Build Sequence

#### Phase 1: Controller Setup

1. Flash the Jetson Orin Nano Super with JetPack 6.x.
2. Install dependencies:

```bash
# System packages
sudo apt update && sudo apt install -y \
    gnuradio gr-osmosdr python3-numpy python3-scipy \
    python3-matplotlib python3-flask libhackrf-dev \
    cmake build-essential

# Python packages
pip3 install soapysdr pysoapy cupy-cuda12x dynamixel-sdk

# Verify CUDA
python3 -c "import cupy; print(cupy.cuda.runtime.getDeviceCount())"
```

3. Connect a single HackRF via USB and verify reception:

```bash
hackrf_transfer -r /dev/null -f 2450000000 -s 20000000 -n 20000000
# Should complete without errors
```

4. Run a basic spectrum sweep to establish baseline:

```python
import SoapySDR
import numpy as np

sdr = SoapySDR.Device(dict(driver="hackrf"))
sdr.setSampleRate(SoapySDR.SOAPY_SDR_RX, 0, 20e6)
sdr.setFrequency(SoapySDR.SOAPY_SDR_RX, 0, 2.45e9)

rxStream = sdr.setupStream(SoapySDR.SOAPY_SDR_RX, SoapySDR.SOAPY_SDR_CF32)
sdr.activateStream(rxStream)

buff = np.zeros(1024, np.complex64)
sr = sdr.readStream(rxStream, [buff], len(buff))
print(f"Read {sr.ret} samples, flags={sr.flags}")

# Compute power spectrum
spectrum = 20 * np.log10(np.abs(np.fft.fftshift(np.fft.fft(buff))))
print(f"Peak power: {np.max(spectrum):.1f} dB")

sdr.deactivateStream(rxStream)
sdr.closeStream(rxStream)
```

#### Phase 2: Multi-SDR Array

5. Connect all three HackRFs. Note: each must be on a separate USB 3.0 root hub, or use a powered USB 3.0 hub with sufficient bandwidth (60 MB/s total for 3× HackRFs at 20 Msps).

6. Assign each SDR a unique serial number:

```bash
hackrf_spiflash -w serial1.bin -d <device_index>
```

7. Implement the `ThreatDetector` class from Section 5.1.5. Verify that all three SDRs produce correlated spectra when exposed to the same test signal (e.g., a Wi-Fi router).

#### Phase 3: Gimbal Assembly

8. **If using Dynamixel servos:**
   - Connect MX-106 servos via U2D2 adapter to Jetson USB
   - Configure IDs (1 = azimuth, 2 = elevation) using Dynamixel Wizard
   - Mount servos in an L-bracket configuration: azimuth servo fixed to base plate, elevation servo mounted on azimuth output horn

9. **If using NEMA 23 steppers:**
   - Wire TMC2209 drivers: STEP, DIR, ENABLE to Jetson GPIO
   - Mount AS5600 magnetic encoders on each motor shaft for closed-loop feedback
   - Fabricate or 3D-print bearing housings for both axes
   - Ensure all wiring is strain-relieved and shielded (stepper noise can interfere with SDRs)

10. Mount a camera (USB webcam or Raspberry Pi Camera v3 via CSI) on the gimbal for visual confirmation of pointing direction.

11. Implement and tune the `GimbalController` PID loop from Section 5.2.4. Start with conservative gains (kp=0.5, ki=0.01, kd=0.1) and increase.

#### Phase 4: Integration

12. Mount the SDR antenna array in a fixed, known geometry (equilateral triangle, 0.5 m sides).

13. Implement the full detect → bearing → gimbal point loop:

```python
class TurretSystem:
    def __init__(self):
        self.detector = ThreatDetector(sdr_array)
        self.gimbal = GimbalController(az_motor, el_motor)
        self.state = "SCANNING"
    
    def run(self):
        self.detector.calibrate_baseline()
        
        while True:
            threat = self.detector.detect_anomalies()
            
            if threat and self.state == "SCANNING":
                bearing = self.detector.estimate_bearing(threat)
                self.state = "TRACKING"
                self.gimbal.point_to(bearing, 0)  # 0 elevation default
                log(f"THREAT DETECTED: {threat['frequency_hz']/1e9:.3f} GHz "
                    f"at bearing {bearing:.1f}°")
            
            elif threat and self.state == "TRACKING":
                bearing = self.detector.estimate_bearing(threat)
                self.gimbal.point_to(bearing, 0)
                
                if self.gimbal.is_on_target(bearing, 0):
                    self.state = "LOCKED"
                    log("TARGET LOCKED")
            
            elif not threat:
                self.state = "SCANNING"
            
            time.sleep(0.01)  # 100 Hz loop
```

14. Test with a known RF source (Wi-Fi router, handheld radio, or signal generator) at various positions. Verify the gimbal tracks the source.

### 7.2 Tier 2 Build Notes

> **Requires FCC Part 5 Experimental Radio Service License before any RF emission.**

15. Mount the DDS eval board inside an RF-shielded enclosure.

16. Connect DDS output → attenuator (for safety) → GaN PA input → waveguide → horn antenna mounted on the gimbal.

17. Initial testing must be performed inside a shielded room or anechoic chamber. Alternatively, use an RF test enclosure (Ramsey boxes) for low-power validation.

18. Add the `generate_jam_waveform()` function from Section 5.4.3 to the engagement logic, triggered only when `state == "LOCKED"`.

### 7.3 Tier 3 Build Notes

**This section intentionally omits step-by-step instructions.** If you have reached the point of federal authorization to build a Tier 3 system, you have access to qualified high-voltage engineers and RF safety officers who will dictate the build procedure. What follows are **design constraints and safety requirements only.**

See Section 9 for legal prerequisites and Section 10 for safety architecture.

---

## 8. Testing & Calibration

### 8.1 Tier 1 Testing Protocol

1. **Baseline stability test:** Run the detector for 24 hours. Verify that the baseline spectrum is stable and that the anomaly detector does not produce false positives from normal ambient sources (WiFi, Bluetooth, microwave ovens, etc.).

2. **Known-source tracking test:** Place a low-power 2.4 GHz transmitter (e.g., a handheld radio or ESP32 dev board transmitting a carrier) at known positions. Verify DOA estimation accuracy to within ±5°.

3. **Gimbal response test:** Command the gimbal to a series of known positions. Verify pointing accuracy with a laser pointer mounted on the gimbal.

4. **Integration test:** Move the test transmitter slowly around the detection array. Verify the gimbal tracks it smoothly without oscillation.

### 8.2 Tier 2 Testing Protocol

All testing must be conducted in a shielded environment.

1. **Emission verification:** Use a calibrated spectrum analyzer to verify that the jammer output is within licensed parameters (frequency, bandwidth, power).

2. **Disruption test:** Use the jammer against a known pulsed RF source and verify disruption of the pulse structure using an oscilloscope with RF demodulation.

3. **Leakage test:** With the jammer operating inside the shielded enclosure, verify zero detectable emissions outside the enclosure using a field probe.

---

## 9. Legal Framework & Controlled Device Thresholds

### 9.1 Federal Communications Commission (FCC)

#### 9.1.1 Receive-Only Systems (Tier 1)

**Fully legal.** No license required to receive and analyze RF signals in any band. This is protected under 47 U.S.C. § 302a and established by decades of precedent in amateur radio, spectrum monitoring, and RF engineering.

#### 9.1.2 Intentional RF Emission (Tier 2)

Any intentional RF emission outside of ISM bands or amateur allocations requires licensing:

- **FCC Part 5 Experimental Radio Service License:** This is the appropriate path for research and development of RF systems. Application is made via the FCC's Electronic Comment Filing System (ECFS) or the Office of Engineering and Technology (OET) Experimental Licensing System (ELS).
- Grants are typically issued for specific frequency ranges, power levels, locations, and time periods.
- Processing time: 30–90 days.
- Cost: $70 filing fee (as of 2025; verify current fee schedule).

**Jamming is broadly illegal under 47 U.S.C. § 333.** The Communications Act of 1934 (as amended) prohibits the operation, marketing, or sale of jamming equipment. **This applies even to self-defense scenarios on your own property.** The only exceptions are for authorized federal government use.

Practical implication: Tier 2 testing must occur in a **shielded enclosure** that prevents any emission into the environment. The experimental license covers the emission inside the enclosure for R&D purposes.

#### 9.1.3 High-Power Directed RF (Tier 3)

A high-power directed RF emitter designed to disrupt or damage electronic systems or cause biological effects is, for regulatory purposes, potentially:

- An **intentional radiator** subject to FCC Part 15/18 limits (far exceeding them)
- A **weapon** if designed to cause harm, subject to state and federal weapons laws
- Subject to **ITAR** (International Traffic in Arms Regulations) if the design or components have military applications and are shared internationally
- Subject to **EAR** (Export Administration Regulations) for high-power microwave components

### 9.2 Department of Defense / Department of Energy

If the system is being developed for or with government entities, it may fall under DOD Directive 3000.09 (Directed Energy Weapons) or DOE regulations for high-energy systems.

### 9.3 State and Local Law

Many states have laws against **electromagnetic weapons** or **electronic harassment devices.** For example:

- **Maine (Title 17-A, §1004):** Prohibits possession of "an electronic weapon"
- **Massachusetts, Michigan, Illinois, and others** have similar statutes
- Check your state's criminal code for terms like "electronic weapon," "directed energy device," or "electromagnetic device"

### 9.4 How to Do This Legally

**Tier 1:** Build freely. Document everything. You are building a **spectrum monitoring and tracking platform**, which is entirely lawful.

**Tier 2:** 
1. Apply for FCC Part 5 Experimental License
2. Specify your frequency range, maximum power (keep it under 1W), location, and purpose
3. Conduct all emissions testing inside a shielded enclosure
4. Maintain logs as required by your license terms

**Tier 3:**
1. **Engage an attorney** specializing in telecommunications law and/or defense contracting
2. If affiliated with a university, work through the university's research compliance office
3. If developing for government use, pursue a **CRADA** (Cooperative Research and Development Agreement) with a relevant federal agency
4. If developing commercially, engage with ITAR/EAR counsel to determine export control classification
5. **Do not assemble the complete system until legal authorization is confirmed in writing**

### 9.5 What Gets You in Trouble

| Action | Risk Level | Consequence |
|--------|-----------|-------------|
| Building Tier 1 (receive-only) | None | Fully legal |
| Emitting <1W in ISM band (2.4 GHz) | Low | Legal under Part 15 if within limits |
| Emitting >1W without license | High | FCC enforcement, fines up to $100k+ |
| Operating a jammer in open air | Very High | Federal crime, fines + imprisonment |
| Building a system designed to harm | Extreme | Federal weapons charges, state charges |
| Exporting HPM technology/designs | Extreme | ITAR/EAR violation, imprisonment |

---

## 10. Safety — How Not to Die

### 10.1 The Three Ways This Kills You

#### 10.1.1 Electrocution / Arc Flash

**This is the primary risk and the most likely way to die building this system.**

At Tier 3 voltages (10–50 kV) and stored energies (100–1000 J), a fault creates an **arc flash event**:

1. **Phase 1 (0–1 ms):** Current flows through ionized air gap. Temperature at arc point reaches 5,000–20,000°C.
2. **Phase 2 (1–10 ms):** Copper conductors vaporize, creating expanding plasma. Pressure wave of 100+ dB.
3. **Phase 3 (10–100 ms):** Molten metal droplets and UV radiation project outward. Clothing ignites.

At 1000 J, the arc flash boundary (where incident energy = 1.2 cal/cm², sufficient to cause second-degree burns) can extend **several feet** from the fault point.

**For comparison:** Electric utility workers wear 40 cal/cm² arc flash suits and maintain strict approach boundaries calculated per IEEE 1584. Your garage does not have these.

#### 10.1.2 RF Burns and Radiation Exposure

At Tier 3 power levels, the power density in front of the antenna exceeds safe exposure limits at distances up to tens of meters:

```
Safe Distance Calculation:
  r_safe = sqrt(P_t × G / (4 × pi × S_limit))
  
  For P_t = 10 kW, G = 100, S_limit = 10 W/m² (FCC public limit):
    r_safe = sqrt(10000 × 100 / (4 × 3.14159 × 10))
    r_safe ≈ 28.2 meters
    
  For occupational limit (50 W/m²):
    r_safe ≈ 12.6 meters
```

RF burns from microwave exposure are **deep tissue burns** — the skin may appear undamaged while subcutaneous fat and muscle are cooked. The lens of the eye is particularly vulnerable (cataracts from chronic sub-thermal exposure).

#### 10.1.3 X-Ray Emission

Magnetrons and high-voltage vacuum tubes can produce **bremsstrahlung X-rays** when electrons strike the anode. At voltages above 15 kV, this becomes a meaningful dose concern. Commercial magnetrons include lead shielding; surplus or custom designs may not.

### 10.2 Mandatory Safety Architecture (Tier 3)

**If you build a Tier 3 system without ALL of the following, you are accepting a meaningful probability of death:**

#### 10.2.1 Energy Dump System

A set of high-power resistors permanently connected through a normally-closed relay across the Marx generator capacitor bank. When power is removed (by a dead-man switch, emergency stop, or safety interlock), the relay closes and dumps all stored energy through the resistors.

```
Dump Resistor Sizing:
  R_dump = V_max / I_max_safe
  P_dump = E_stored / t_dump
  
  For E = 1000 J dumped in 1 second:
    P_dump = 1000 W → need 1 kW+ wirewound resistors
  
  Time constant:
    τ = R × C_total
    
  After 5τ, energy is reduced to <1% of initial.
  Choose R such that 5τ < 5 seconds.
```

#### 10.2.2 Dead-Man Switch

The system must require **continuous operator input** to remain energized. Releasing the switch de-energizes the HV supply and triggers the energy dump. This is a hardware interlock — not software.

#### 10.2.3 Physical Barriers

The Marx generator and PFN must be enclosed in a **polycarbonate or HDPE enclosure** with grounded conductive mesh on the exterior (Faraday cage) and no openings large enough to admit a finger (IP2X minimum per IEC 60529).

#### 10.2.4 Grounding

All chassis, enclosures, waveguide, and antenna structures must be bonded to a common ground bus connected to building earth ground with <1Ω impedance. Use 6 AWG or larger copper ground conductors.

#### 10.2.5 Lockout/Tagout (LOTO)

Before any maintenance:

1. De-energize the system
2. Wait for dump cycle to complete (monitor voltage with a HV probe — do NOT rely on timers)
3. Apply a **grounding stick** (insulated rod with grounded conductor) to every capacitor terminal
4. Verify zero voltage with a calibrated HV probe
5. Apply physical locks to all HV disconnect points
6. Only then approach the system

#### 10.2.6 RF Safety Zone

Establish and mark an exclusion zone around the antenna based on the safe distance calculation in 10.1.2. During operation, **no personnel** within this zone. Particular attention to the **main beam direction** where power density is highest.

#### 10.2.7 Eye Protection

Polycarbonate safety glasses block incidental RF heating of the cornea but do not protect against high-intensity mainbeam exposure. During Tier 3 operation, the antenna must never point toward occupied areas — **the eye is the most vulnerable organ** to microwave exposure.

#### 10.2.8 Monitoring

An independent RF field probe (e.g., Narda NBM-550) should be positioned at the boundary of the exclusion zone during any emission test, with a threshold alarm set at 50% of the occupational exposure limit.

### 10.3 Personal Protective Equipment Matrix

| PPE | Tier 1 | Tier 2 | Tier 3 |
|-----|--------|--------|--------|
| Safety glasses | Recommended | Required | Required |
| ESD wrist strap | Recommended | Recommended | — (lethal at HV) |
| HV-rated gloves (Class 0+) | — | — | **Required** |
| Arc flash suit (40 cal/cm²) | — | — | **Required during assembly** |
| Face shield | — | — | **Required** |
| RF monitoring badge | — | Recommended | **Required** |

> **CRITICAL: Never wear an ESD wrist strap when working with Tier 3 voltages.** The wrist strap creates a direct low-resistance path from the HV through your body to ground. At 30 kV, this is instantly lethal.

### 10.4 The Buddy Rule

**Never work on a Tier 3 system alone.** A second person must be present, out of reach of the equipment, with access to a phone and knowledge of how to shut down the system and administer first aid for electrical burns. Both persons must know the location of the nearest AED (automated external defibrillator).

---

## 11. Future Work

### 11.1 Machine Learning Threat Classification

The current anomaly detector uses statistical deviation from a baseline spectrum. A trained classifier (CNN or transformer on spectrogram data) could distinguish threat signals from benign anomalies (radar, communications, WiFi) with higher specificity. Training data could be generated synthetically or collected in a controlled environment.

### 11.2 Phased Array Emitter

Replacing the mechanically steered horn antenna with an electronically steered phased array eliminates gimbal latency entirely. S-band phased arrays using commodity WiFi PA chips (e.g., Skyworks SE5516) are feasible at Tier 2 power levels.

### 11.3 Distributed Sensor Network

Multiple Tier 1 nodes networked via Ethernet or WiFi, with centralized processing on a server, would provide facility-wide coverage and triangulation accuracy superior to a single-point array.

### 11.4 Integration with Physical Shielding

The turret concept is complementary to passive shielding (conductive paint, metallized window film). A combined system uses passive shielding to reduce incident power and the active turret to detect and potentially disrupt the source.

### 11.5 Counter-UAS Application

The same gimbal + emitter platform could be repurposed for counter-drone operations, disrupting command links or GPS receivers on small UAS. This application is subject to additional FAA and FCC regulations.

---

## 12. References

1. National Academies of Sciences, Engineering, and Medicine. (2020). *An Assessment of Illness in U.S. Government Employees and Their Families at Overseas Embassies.* Washington, DC: The National Academies Press.

2. Lin, J.C. (1978). *Microwave Auditory Effects and Applications.* Springfield, IL: Charles C Thomas.

3. Balanis, C.A. (2016). *Antenna Theory: Analysis and Design,* 4th ed. Wiley.

4. Pozar, D.M. (2011). *Microwave Engineering,* 4th ed. Wiley.

5. Mesyats, G.A. (2005). *Pulsed Power.* New York: Kluwer Academic/Plenum.

6. IEEE 1584-2018. *IEEE Guide for Performing Arc-Flash Hazard Calculations.*

7. NFPA 70E-2021. *Standard for Electrical Safety in the Workplace.*

8. FCC Part 5 — Experimental Radio Service. 47 CFR Part 5.

9. FCC OET Bulletin 65 — *Evaluating Compliance with FCC Guidelines for Human Exposure to Radiofrequency Electromagnetic Fields.*

10. ITAR — International Traffic in Arms Regulations. 22 CFR Parts 120–130.

11. Benford, J., Swegle, J.A., & Schamiloglu, E. (2015). *High Power Microwaves,* 3rd ed. CRC Press.

12. Rahmat-Samii, Y. & Michielssen, E. (1999). *Electromagnetic Optimization by Genetic Algorithms.* Wiley.

---

## 13. Appendices

### Appendix A: Glossary

| Term | Definition |
|------|-----------|
| **DOA** | Direction of Arrival — the angle from which an RF signal originates |
| **DDS** | Direct Digital Synthesizer — generates precise RF waveforms digitally |
| **GaN** | Gallium Nitride — semiconductor material for high-efficiency RF power amplifiers |
| **HPM** | High Power Microwave |
| **ISM** | Industrial, Scientific, and Medical — unlicensed RF bands (e.g., 2.4 GHz) |
| **ITAR** | International Traffic in Arms Regulations |
| **LOTO** | Lockout/Tagout — safety procedure for de-energizing equipment |
| **Marx Generator** | Voltage multiplier using parallel-charged, series-discharged capacitors |
| **PFN** | Pulse Forming Network — shapes discharge pulse for magnetron drive |
| **PRF** | Pulse Repetition Frequency — number of pulses per second |
| **SAR** | Specific Absorption Rate — RF power absorbed per unit mass of tissue |
| **SDR** | Software Defined Radio |
| **TDOA** | Time Difference of Arrival — technique for geolocating RF emitters |
| **Vircator** | Virtual Cathode Oscillator — simple HPM source |

### Appendix B: Quick Reference — "Am I About to Break the Law?"

```
START
  │
  ├─ Am I only receiving RF signals?
  │   YES → Legal. Proceed.
  │   NO ↓
  │
  ├─ Am I emitting RF?
  │   YES ↓
  │
  ├─ Is my power < 1W in ISM band (2.4 GHz, 5.8 GHz)?
  │   YES → Likely legal under Part 15. Check specific limits.
  │   NO ↓
  │
  ├─ Do I have an FCC Experimental License for this frequency/power?
  │   YES → Legal within license terms. Proceed.
  │   NO ↓
  │
  ├─ Am I emitting in open air (not a shielded enclosure)?
  │   YES → STOP. Illegal. Get a license.
  │   NO ↓
  │
  ├─ Is my system designed to disrupt, damage, or harm?
  │   YES → STOP. Consult attorney. Federal weapon/jamming laws apply.
  │   NO ↓
  │
  └─ Proceed with caution. Document everything.
```

### Appendix C: Emergency Procedures

**Electrical Contact / Arc Flash:**
1. Do NOT touch the victim if they are still in contact with the circuit
2. De-energize the system using the emergency stop (if safe to reach)
3. Call 911
4. If victim is unresponsive and not breathing, begin CPR
5. If AED is available, apply it — follow voice prompts
6. For burns: cool with running water, do not apply ice or ointments
7. Treat for shock: lay flat, elevate legs, keep warm

**RF Exposure:**
1. Move away from the antenna/emitter immediately
2. If eye exposure suspected: seek ophthalmological evaluation within 24 hours
3. For skin burns: treat as thermal burns — cool with water, seek medical attention
4. Report the exposure and document the estimated power density and duration

---

*This document is a living reference. Contributions, corrections, and safety improvements are welcome. If you find an error in the safety procedures, please report it immediately.*

*"To be made in the image of the Creator is to be a creator of very good things yourself." Build responsibly.*
