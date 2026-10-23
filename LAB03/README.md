# ACUS Lab 3 — Frequency Domain: FFT & Spectrograms

---

**Course**: ACUS - Exploració, Acústica Marina i Sistemes de Sonar
**Program**: Ciències i Tecnologies del Mar
**Author**: Enoc Martínez
**Department**: Departament d'Enginyeria (EEL)
**Contact**: enoc.martinez@upc.edu

<p align="center">
  <img height="100" src="https://github.com/EnocMartinez/citm-acus-2026/blob/main/resources/banner.png?raw=true" alt="infographic">
</p>

---

## Introduction

A humpback song is built from **phrases that repeat**. Today you will measure
every phrase in a real song automatically, and use those measurements to work
out the song's structure — which is how whale song was first understood, by
Payne and McVay in 1971.

You cannot do this by looking at pictures. You need numbers.

**Build your own script.** There is no file to download: create `LAB03.py`
yourself and copy each code block into it, in order. Where you see `____`, fill
in the blank. 📝 marks a question to answer in your report.

### What to hand in

* A report in PDF with snapshots of your code for every task.
* The answers to 📝 **Q1 – Q7**.
* An `AI usage` section describing which tools you used and why.
* Your script, in a single file named `LAB03.py`.

### The recording

`humpback_song.wav` — 16 s of humpback whale song, mono, 22050 Hz, 16-bit.
Five phrases, with silence between them.

---

## Task 0 — Listen first

**No Python.** Open `humpback_song.wav` in Audacity and play it. Switch the
track to spectrogram view (click the track name → *Spectrogram*) and watch it.

**Write down on paper:** how many phrases you hear, and which ones sound like
repeats. You will check this guess in Task 6, so it must appear in your report.

---

## Task 1 — Load and plot

Start your script with these imports, then load the file.

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from scipy.io import wavfile
from scipy.signal import spectrogram

FILENAME = "humpback_song.wav"

PHRASES = [
    ("P1",  0.56,  1.30),
    ("P2",  3.62,  5.15),
    ("P3",  7.17,  7.92),
    ("P4",  9.37, 11.03),
    ("P5", 12.59, 13.73),
]
```

```python
fs, data = wavfile.read(FILENAME)

n_samples = ____(data)              # how many samples? (Lab 1)
duration  = ____ / ____             # samples and sampling rate
t = np.arange(n_samples) / ____     # time axis, t = n / fs

print(f"sampling rate : {fs} Hz")
print(f"duration      : {duration} s")

plt.figure(figsize=(12, 4))
plt.plot(t, data, lw=0.3)
plt.xlabel("time (s)")
plt.ylabel("amplitude (counts)")
plt.title("Humpback whale song")
plt.tight_layout()
plt.show()
```

You should see five bursts separated by quieter gaps. Those are the phrases,
and their start and end times are the `PHRASES` list above.

> 📝 **Q1** This file was recorded at 22050 Hz. What is the highest frequency it
> can represent, and why? (Session 4.)

---

## Task 2 — One FFT for the whole song

`spectrum_of` returns the list of frequencies and how much of each is present.
`np.argmax` gives the **position** of the largest value (Lab 1), which you use
to pick out the matching frequency.

```python
def spectrum_of(x, fs):
    x = x.astype(float)
    N = len(x)
    mag = np.abs(np.fft.fft(x)) / N
    f = np.fft.fftfreq(N, 1 / fs)
    return f, mag


f_all, mag_all = spectrum_of(data, fs)

loudest = f_all[np.____(mag_all)]          # the loudest frequency

print(f"loudest frequency in the whole song: {loudest:.1f} Hz")

mask = (f_all >= 0) & (f_all <= 2000)   # True/False, one per frequency
f_subset = f_all[mask]                  # keep only the True ones
m_subset = mag_all[mask]

plt.figure(figsize=(10, 4))
plt.plot(f_subset, m_subset, lw=0.7)
plt.xlabel("frequency (Hz)")
plt.ylabel("magnitude")
plt.title("One FFT of the whole 16-second song")
plt.tight_layout()
plt.show()
```

The `mask` line keeps only the 0–2000 Hz part, where the whale is. It builds a
`True`/`False` list with one entry per frequency, and `f_all[mask]` keeps the
`True` ones. **Copy lines like this as they are** — you will not need to write
them yourself.

> 📝 **Q2** You have just described a 16-second song, containing five different
> phrases, with **one number**. What information did that throw away?

---

## Task 3 — One FFT per phrase

To cut a piece out of the recording you need sample numbers, not seconds:
sample `n = time(s) * Fs(Hz)`.

The figure has two rows: the phrase itself on top, its spectrum underneath.
`axes` is now a grid, so you pick one panel with `axes[row, column]`.

```python
fig, axes = plt.subplots(2, len(PHRASES), figsize=(18, 6), sharey="row")

for i in range(len(PHRASES)):
    name  = PHRASES[i][0]          # "P1", "P2", ...
    start = PHRASES[i][1]          # start time, in seconds
    end   = PHRASES[i][2]          # end time, in seconds
    ax_time = axes[0, i]           # top row:    the i-th waveform
    ax_freq = axes[1, i]           # bottom row: the i-th spectrum

    n0 = int(start * ____)                 # seconds -> sample number
    n1 = int(end * ____)
    phrase = data[n0:n1]
    t_phrase = np.arange(n0, n1) / fs      # time axis for this phrase

    f_p, mag_p = spectrum_of(____, fs)     # the spectrum of this phrase
    loudest_f = f_p[np.argmax(mag_p)]      # its loudest frequency

    print(f"-> {name} : {loudest_f:.1f} Hz")

    mask = (f_p >= 0) & (f_p <= 2000)  # boolean mask, keep only freq between 0 and 2 kHz
    f_subset = f_p[mask]               # apply the mask to frequencies
    m_subset = mag_p[mask]             # apply the mask to magnitudes

    ax_time.plot(t_phrase, phrase, lw=0.3)  # time vs amplitude plot
    ax_time.set_title(name)
    ax_time.set_xlabel("time (s)")

    ax_freq.plot(____, ____)  # frequency vs magnitude plot (Fourier)
    ax_freq.set_title(f"{name} peak {loudest_f:.0f} Hz")
    ax_freq.set_xlabel("frequency (Hz)")

axes[0, 0].set_ylabel("amplitude (counts)")
axes[1, 0].set_ylabel("magnitude")
plt.tight_layout()
plt.show()
```

Five numbers instead of one. Compare the two rows: the waveforms are all much
the same noisy blob, while the spectra clearly differ. But two of these phrases
**rise in pitch** from beginning to end, and one holds roughly the same note.

> 📝 **Q3** Looking only at this figure, can you tell which phrases rise and
> which stays flat? Explain why or why not.

---

## Task 4 — The spectrogram

A spectrogram is the recording cut into short slices, with one FFT per slice.
It returns three things:

| | |
|---|---|
| `f` | the frequencies — one per **row** |
| `t_spec` | the time slices — one per **column** |
| `S` | the power at each frequency, in each slice |

```python
phrase1 = data[int(0.56 * fs):int(1.30 * fs)]

f, t_spec, S = spectrogram(phrase1, fs=fs, nperseg=1024, noverlap=768)

print(f"f      : {len(f)} frequencies, {f[0]:.0f} to {f[-1]:.0f} Hz")
print(f"t_spec : {len(t_spec)} time slices")
print(f"S      : shape {S.shape} = (frequencies, time slices)")
print(f"step between frequencies : {f[1] - f[0]:.1f} Hz")
print(f"step between time slices : {1000 * (t_spec[1] - t_spec[0]):.1f} ms")
print(S[0:4, 0:4])

power = S[____, ____]      # a table is indexed [row, column]
print(f"power at {f[20]:.0f} Hz, at t = {t_spec[10]:.3f} s : {power:.4g}")

plt.figure(figsize=(10, 4))
plt.pcolormesh(t_spec, f, 10 * np.log10(S + 1e-12), shading="auto", cmap="magma")
plt.ylim(0, 2000)
plt.xlabel("time (s)")
plt.ylabel("frequency (Hz)")
plt.title("Spectrogram of phrase P1")
plt.colorbar(label="power (dB, relative)")
plt.tight_layout()
plt.show()
```

`S` is a table of 513 rows × 60 columns. Each number says how much power there
was, at one frequency, during one slice of time. The picture is that same table,
with colour instead of height.

> 📝 **Q4** Include this figure. Does P1 rise, fall, or stay flat? Why could you
> not answer this in Task 3?

---

## Task 5 — Measure the contour

`axis=0` means **look down each column**, giving one answer per column:

```python
toy = np.array([[1.0, 9.0, 2.0],
                [8.0, 3.0, 4.0],
                [2.0, 5.0, 7.0]])
print(toy)
print(np.argmax(toy, axis=0))       # -> [1 0 2]
```

Column 0 is won by row 1 (value 8), column 1 by row 0 (value 9), column 2 by
row 2 (value 7).

The whale is between 100 and 2000 Hz; outside that there is only noise. The
`band` line below builds a True/False list, one per frequency, and `f[band]`
keeps only the True ones. **Copy it as it is** — you will not need to write
lines like this yourself.

```python
FMIN = 100
FMAX = 2000
band = (f >= FMIN) & (f <= FMAX)

f_band = f[band]
S_band = S[band]

row_of_winner = np.argmax(S_band, axis=____)    # loudest row of EVERY column
contour = f_band[row_of_winner]

print(f"contour has {len(contour)} values, one per time slice")

plt.figure(figsize=(10, 4))
plt.pcolormesh(t_spec, f, 10 * np.log10(S + 1e-12), shading="auto", cmap="magma")
plt.plot(t_spec, contour, color="cyan", lw=2, label="measured contour")
plt.ylim(0, 2000)
plt.xlabel("time (s)")
plt.ylabel("frequency (Hz)")
plt.title("P1 with the measured contour on top")
plt.legend()
plt.tight_layout()
plt.show()
```

This is the same question you asked in Task 2 — *which frequency is loudest?* —
asked once per time slice instead of once for the whole phrase. The answer is a
list of 60 frequencies: where the whale was, moment by moment. That is the
**contour**.

> 📝 **Q5** Include this figure. Does the cyan line follow the whale?

---

## Task 6 — Measure all five, and find the structure

`measure_phrase` packs Tasks 4 and 5 into one function: give it a start and end
time, it returns five numbers describing that phrase — how long it lasts, where
it starts and ends, and its lowest and highest frequency.

```python
def measure_phrase(data, fs, start, end):
    phrase = data[int(start * fs):int(end * fs)]
    f, t_spec, S = spectrogram(phrase, fs=fs, nperseg=1024, noverlap=768)

    band = (f >= FMIN) & (f <= FMAX)
    contour = f[band][np.argmax(S[band], axis=0)]

    duration = ____ - ____        # length of the phrase, in seconds
    f_start  = contour[____]      # first value of the contour (Lab 1)
    f_end    = contour[____]      # last value
    f_min    = contour.____()     # lowest frequency reached (Lab 1)
    f_max    = contour.____()     # highest frequency reached

    return duration, f_start, f_end, f_min, f_max
```

Now call it once per phrase, collect the results into a table, and plot one
point per phrase.

```python
rows = []
for i in range(len(PHRASES)):
    name  = PHRASES[i][0]
    start = PHRASES[i][1]
    end   = PHRASES[i][2]

    duration, f_start, f_end, f_min, f_max = measure_phrase(data, fs, start, end)

    rows.append({"phrase": name, "duration": duration, "f_start": f_start,
                 "f_end": f_end, "f_min": f_min, "f_max": f_max})

df = pd.DataFrame(rows)
df = df.round({"duration": 2, "f_start": 0, "f_end": 0, "f_min": 0, "f_max": 0})
print(df)

plt.figure(figsize=(7, 5.5))
plt.scatter(df["duration"], df["f_start"], s=90)
for i in range(len(df)):
    plt.annotate(df["phrase"][i], (df["duration"][i], df["f_start"][i]),
                 textcoords="offset points", xytext=(8, 5))
plt.xlabel("duration (s)")
plt.ylabel("start frequency (Hz)")
plt.title("One point per phrase")
plt.grid(alpha=0.3)
plt.tight_layout()
plt.show()
```

One row per phrase, one point per phrase. Phrases of the same type land near
each other on the scatter plot.

> 📝 **Q6** Include your table and the scatter plot. Which phrases group
> together? Give each group a letter and write the song out in order, like
> `A B A B C`. Does it match the guess you made by ear in Task 0? The phrases in
> a group are similar but not identical — give one number that differs between
> two of them.

---

## Task 7 — Putting it all together

Last step: the Task 3 figure again, but with a third row added — the
spectrogram of each phrase, with its contour on top. Everything in this block
has already appeared somewhere above. If Tasks 1–6 worked, copy-pasting this
should just work too.

```python
fig, axes = plt.subplots(3, len(PHRASES), figsize=(18, 9), sharey="row")

for i in range(len(PHRASES)):
    name  = PHRASES[i][0]
    start = PHRASES[i][1]
    end   = PHRASES[i][2]
    ax_time = axes[0, i]
    ax_freq = axes[1, i]
    ax_spec = axes[2, i]

    n0 = int(start * fs)
    n1 = int(end * fs)
    phrase = data[n0:n1]
    t_phrase = np.arange(n0, n1) / fs

    f_p, mag_p = spectrum_of(phrase, fs)
    loudest_f = f_p[np.argmax(mag_p)]
    mask = (f_p >= 0) & (f_p <= 2000)
    f_subset = f_p[mask]
    m_subset = mag_p[mask]

    f, t_spec, S = spectrogram(phrase, fs=fs, nperseg=1024, noverlap=768)
    band = (f >= FMIN) & (f <= FMAX)
    contour = f[band][np.argmax(S[band], axis=0)]

    ax_time.plot(t_phrase, phrase, lw=0.3)
    ax_time.set_title(name)
    ax_time.set_xlabel("time (s)")

    ax_freq.plot(f_subset, m_subset)
    ax_freq.set_title(f"peak {loudest_f:.0f} Hz")
    ax_freq.set_xlabel("frequency (Hz)")

    ax_spec.pcolormesh(t_spec, f, 10 * np.log10(S + 1e-12), shading="auto", cmap="magma")
    ax_spec.plot(t_spec, contour, color="cyan", lw=1.5)
    ax_spec.set_ylim(0, 2000)
    ax_spec.set_xlabel("time (s)")

axes[0, 0].set_ylabel("amplitude (counts)")
axes[1, 0].set_ylabel("magnitude")
axes[2, 0].set_ylabel("frequency (Hz)")
plt.tight_layout()
plt.show()
```

> 📝 **Q7** Include this figure. Pick one phrase and describe, in one sentence
> each, what row 1, row 2 and row 3 tell you about it.

---

## Wrap-up

Today you learned how to:

- compute a **spectrum** with the FFT and find its loudest frequency
- see why one FFT cannot describe a sound that changes — it knows which
  frequencies, not when
- compute a **spectrogram**, and read it as a table of numbers as well as a picture
- measure a call's **contour** automatically, by asking "which frequency is
  loudest?" once per time slice
- turn a recording into a table of measurements, and find the repeated structure
  of a whale song
- read a waveform, a spectrum and a spectrogram side by side, and see how each
  one adds information the others do not have
