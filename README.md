# Timing Side-Channel Measurements on ESP32 and ESP8266

Raw experimental data collected by **Abrar Sobhan** (2026) for a study of how the position of the first mismatching byte affects comparison timing on ESP32 and ESP8266 microcontrollers.

This is a **data-only release**. The four CSV files were copied without modification from the original research archive. Firmware, host-side collection scripts, and manuscript drafts are **not** included here.

## Files

| File | Platform | Measurement condition | Rows |
| --- | --- | --- | ---: |
| `data/esp32_prefix_timing_data.csv` | ESP32 | Interrupt-protected | 10,200 |
| `data/esp32_prefix_timing_data1unprotected.csv` | ESP32 | Unprotected | 10,200 |
| `data/esp8266_protected_data.csv` | ESP8266 | Interrupt-protected | 10,200 |
| `data/esp8266_unprotected_data.csv` | ESP8266 | Unprotected | 10,200 |
| **Total** | | | **40,800** |

The filenames are preserved from the original acquisition files, including the `data1unprotected` suffix.

## Experimental design

- Each comparison operates on a **16-byte input** and a fixed 16-byte reference secret.
- `V` is an **early-exit comparison**, returning as soon as a mismatching byte is found.
- `C` is a **fixed-work comparison** that evaluates all 16 byte pairs and combines the XOR differences using a bitwise OR accumulator. *Fixed-work* describes the implementation strategy; it is not a formal guarantee of constant-time machine code.
- `prefix_match_len` takes each integer value from **0 through 16**. For values 0–15, input generation ensures that exactly that many leading bytes match and the next byte differs. At 16, all bytes match.
- For each platform/condition file, **300 observations** were recorded at each of the 17 prefix lengths for each of the two comparison modes: `17 × 300 × 2 = 10,200` rows.
- Both comparison functions were annotated with `noinline` and GCC `optimize("O0")` in the acquisition firmware. The character in `O0` is a capital letter **O**, followed by the numeral **0**.
- For the protected ESP32 condition, the timed section was enclosed in a FreeRTOS critical section and used compiler memory barriers; on the ESP8266, interrupts were disabled around the timed section with analogous barriers. For the unprotected conditions, interrupts were not disabled around the measured comparison.

## Data dictionary

All four UTF-8 CSV files share these columns:

| Column | Meaning |
| --- | --- |
| `trial_id` | Sequential row identifier within a file, beginning at 0; not a global identifier across devices. |
| `mode` | `V` = early-exit comparison; `C` = fixed-work XOR/OR comparison. |
| `prefix_match_len` | Number of correctly matching leading bytes, from 0 to 16 inclusive. |
| `match_result` | Boolean comparison result reported by the device: `0` = mismatch, `1` = complete match. |
| `cpu_cycles` | Raw elapsed CPU cycle-counter difference for the measured comparison call, including measurement overhead. This is a **cycle count**, not milliseconds or microseconds. |

The ESP32 measurements used `esp_cpu_get_cycle_count()`; the ESP8266 measurements used `ESP.getCycleCount()`.

## Data integrity and interpretation

- All **40,800 raw rows** are retained; no rows were filtered or modified for this release.
- Each file contains 5,100 observations per mode, 300 for every `(mode, prefix_match_len)` combination, and no missing prefix-length categories.
- Mode `V` was collected before mode `C`, and prefix lengths were swept in ascending order. This fixed acquisition order should be considered when evaluating possible time-dependent effects.
- Some observations have unusually high cycle counts; these are left intact in the raw data rather than being silently discarded.
- The CSVs do not record device serial numbers, board/core revisions, compiler build logs, source commit identifiers, individual input vectors, device temperatures, or independent measurement timestamps. Those details cannot be reconstructed from the CSVs alone.
- Comparisons across devices reflect different hardware and timing environments. These data alone do not establish a device-independent leakage rate, a practical secret-recovery attack, or a formal constant-time security proof.

## Suggested analyses

Group by `mode` and `prefix_match_len`, calculate median `cpu_cycles` and dispersion (for example, interquartile ranges), and inspect the full distribution rather than relying solely on means. When comparing protected and unprotected runs, keep each device and acquisition condition separately identified.

## Citation and reuse

Suggested provisional citation: **Sobhan, A. (2026). *Timing side-channel measurements of early-exit and fixed-work comparisons on ESP32 and ESP8266* [Dataset]. GitHub.** Add the repository URL or an archival DOI once available.

No reuse license has been selected yet. Public availability should not be interpreted as permission to redistribute or adapt the files under a particular license. The author may add a license in a later release.
