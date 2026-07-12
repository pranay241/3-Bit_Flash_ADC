# 3-Bit Flash ADC

A mathematically rigorous design, analysis, and implementation of a 3-bit
Flash Analog-to-Digital Converter — covering the full signal chain from
sampling theory through quantization, the parallel comparator hardware,
and measured performance against theoretical predictions.

## Why Flash ADC

Flash (parallel) architecture trades resolution for speed: instead of
successive approximation or integration, it uses `2^N − 1` comparators
firing simultaneously against a reference ladder, making it the fastest
ADC topology at the cost of exponentially more hardware per added bit.
This project builds and analyzes the 3-bit case end to end.

## Signal chain covered

```
Continuous signal x(t)
        │
        ▼
Sample & Hold (S&H)
        │
        ▼
Resistive reference ladder → comparator array (quantization)
        │
        ▼
Thermometer code → Priority encoder (binary encoding)
        │
        ▼
Digital output b2 b1 b0
```

### 1. Sampling & aliasing
Formalizes sampling as multiplication with a Dirac comb, derives the
sampled spectrum via the convolution theorem, and applies the
Nyquist–Shannon criterion (`fs > 2·fmax`) to characterize when aliasing
occurs and what anti-aliasing filtering is required.

### 2. Sample & Hold dynamics
Models the S&H stage as an RC circuit: derives acquisition time for a
given accuracy target (`t_acq ≈ 6.9·R_on·C` for 0.1% accuracy), hold-mode
droop from leakage current, and aperture (clock jitter) error for a
sinusoidal input.

### 3. Quantization
For an N-bit converter, the input range is split into `2^N` intervals of
width `Δ = Vref / 2^N`. Derives the bound on quantization error, its
variance (`Δ²/12` for uniform input), and the resulting theoretical SQNR:

```
SQNR(dB) = 6.02·N + 1.76
```

### 4. Reference ladder & comparator array
- **Ladder network**: Thevenin-equivalent resistance and tap voltage at
  each of the 8 nodes, plus sensitivity of each tap voltage to individual
  resistor tolerance.
- **Comparators**: models each comparator's input-referred offset as
  Gaussian noise and derives the resulting differential nonlinearity
  (DNL) distribution.

### 5. Priority encoder
Converts the 7-bit thermometer code `T1..T7` into binary output
`B2 B1 B0` via Boolean logic, with a Karnaugh-map-minimized version
that also corrects for "bubble errors" (non-monotonic comparator firing
due to offset/noise).

## Experimental setup

- **Input**: 1 kHz sinusoid, `Vin(t) = 2.5 + 2.5·sin(2π·1000·t)` V
- **Sampling rate**: 10 kHz (satisfies Nyquist, `fs/fin = 10`)
- **Expected LSB**: `Δ = 5V / 2³ = 0.625 V`
- **Reconstruction**: R-2R DAC output verified against the binary code

## Results — theoretical vs. measured

| Parameter | Theoretical | Measured | Error |
|---|---|---|---|
| LSB (Δ) | 0.625 V | 0.66 V | 5.6% |
| Offset error | 0 V | 200 mV | — |
| Gain error | 0% | 2.2% | — |
| Conversion time | < 1 µs | 128.4 µs | — |
| DNL | 0 LSB | 0.4 LSB | — |
| INL | 0 LSB | 0.8 LSB | — |

From these, Total Unadjusted Error (TUE) was computed and used to derive
**SINAD ≈ 15.8 dB** and **ENOB ≈ 2.33 bits** — below the theoretical
3-bit maximum, attributed mainly to the measured comparator/ladder offset
and gain error.

Conversion time breakdown showed the DAC stage (`t_DAC`) dominating the
total ~128 µs, pointing to the R-2R network's time constant as the main
speed bottleneck rather than the comparator or encoder stages.

## Key findings

- Verified the Nyquist sampling criterion experimentally, including
  observable aliasing when `fs < 2·fmax`.
- Theoretical SQNR for 3 bits (19.82 dB) dropped to a measured 15.8 dB
  SINAD due to real-world non-idealities (offset, gain error, DNL/INL).
- The statistical error model (Gaussian comparator/resistor offsets)
  correctly predicted the shape of the observed DNL/INL distribution.

## Possible extensions

- Monte Carlo simulation of resistor/capacitor tolerance effects on
  overall output distribution.
- Information-theoretic channel capacity analysis (`C = fs·log2(1+SNR)`).
- Joint power–area–delay optimization subject to a minimum ENOB and
  Nyquist-rate constraint.

## References

1. Oppenheim, A. V., & Schafer, R. W. — *Discrete-Time Signal Processing*
2. Razavi, B. — *Principles of Data Conversion System Design*
