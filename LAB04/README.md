# ACUS Lab 4 — Filters: Keeping the Frequencies You Want

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

A filter is a frequency amplifier: it doesn't know what a whistle is, or what
noise is, it only knows frequencies — and it lets some through while it
blocks the rest.

* What it lets through is the **pass-band**; what it blocks is the
  **rejection band**.
* The **cutoff** is where you tell it to switch between the two.
* A real filter can't switch instantly — the **order** controls how sharp
  that switch is. Higher order = closer to an ideal cliff, but more
  computation.
* `scipy.signal.butter` designs the filter; `sosfiltfilt` applies it
  **without shifting the signal in time** (useful — it means a feature at
  t = 2.3 s stays at t = 2.3 s after filtering).

Today you build three filters to split a signal into its pieces, then use
one to clean a real, noisy recording.

### What to hand in

* A report in PDF with snapshots of your code for every task.
* The answers to 📝 **Q1 – Q5**.
* An `AI usage` section describing which tools you used and why.
* Your script, in a single file named `LAB04.py`.
* The cleaned `.wav` file you produce in Task 4.

### The recording

`02_toothed_whale_dolphin_whistle.wav` — a real hydrophone recording, 4 s,
mono, 50000 Hz, 16-bit. A dolphin whistle is in there — but the loudest thing
in the recording is a low-frequency hum that is not the dolphin.

---

## Task 0 — Listen first

**No Python.** Open the file in Audacity, play it, and switch to spectrogram
view. The hum is obvious. The whistle is much fainter — you may need to look
more than listen.

**Write down on paper:** roughly where (in Hz) the hum sits, and roughly
where the whistle sits. You will use this in Task 3.

---

## Toolbox — read this before you start

Four building blocks you will reuse all lab. Nothing to fill in here, just
run them and look at the output.

### 1. Design a filter, plot its response

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.io import wavfile
from scipy.signal import butter, sosfiltfilt, sosfreqz, spectrogram

fs_demo = 20000
sos = butter(6, 600, btype="lowpass", fs=fs_demo, output="sos")

w, h = sosfreqz(sos, worN=4000, fs=fs_demo)
mag_db = 20 * np.log10(np.abs(h) + 1e-12)

plt.figure(figsize=(8, 4))
plt.plot(w, mag_db)
plt.axvline(600, color="grey", ls="--", label="cutoff")
plt.axhline(-3, color="red", ls=":", lw=1, label="-3 dB")
plt.xlabel("frequency (Hz)")
plt.ylabel("gain (dB)")
plt.ylim(-80, 5)
plt.title("Low-pass filter, order 6, cutoff 600 Hz")
plt.legend()
plt.tight_layout()
plt.show()
```

A gain of 0 dB is "fully passed", and the further negative it gets the more
blocked that frequency is — this is the same dB scale from earlier sessions,
just applied to a filter's gain instead of a sound level.

### 2. Create and apply a filter, in one function

```python
def lp_filter(data, cutoff, fs, order=6):
    sos = butter(order, cutoff, btype="lowpass", fs=fs, output="sos")
    data_filtered = sosfiltfilt(sos, data)
    return data_filtered
```

Copy this and adapt it for the other two shapes — only `btype` and the
`cutoff` argument change:

```python
def hp_filter(data, cutoff, fs, order=6):
    sos = butter(order, cutoff, btype="highpass", fs=fs, output="sos")
    data_filtered = sosfiltfilt(sos, data)
    return data_filtered


def bp_filter(data, cutoff, fs, order=6):
    sos = butter(order, cutoff, btype="bandpass", fs=fs, output="sos")
    data_filtered = sosfiltfilt(sos, data)
    return data_filtered
```

A band-pass takes **two** cutoffs, not one — call it as
`bp_filter(data, [1000, 2000], fs)`, a list with the low edge and the high
edge of the window you want to keep. `lp_filter` and `hp_filter` take a
single number instead.

`sosfiltfilt` applies the filter **without shifting the signal in time**
(useful — it means a feature at t = 2.3 s stays at t = 2.3 s after
filtering).

### 3. The spectrum, with `rfft`

A real signal's spectrum is a mirror image around 0 Hz — the negative half
carries no information the positive half doesn't already have. `rfft`
("real FFT") skips the mirror and returns only the positive half directly:

```python
def spectrum_of(x, fs):
    x = x.astype(float)
    N = len(x)
    mag = np.abs(np.fft.rfft(x)) / N
    f = np.fft.rfftfreq(N, 1 / fs)
    return f, mag
```

Use it exactly like Lab 3's `spectrum_of` — call it, then plot `f` against
`mag` — except there is no mask to apply: every frequency `rfft` returns is
one you want to keep.

### 4. The spectrogram (recap from Lab 3)

```python
f, t_spec, S = spectrogram(x, fs=fs, nperseg=1024, noverlap=768)

plt.figure(figsize=(10, 4))
plt.pcolormesh(t_spec, f, 10 * np.log10(S + 1e-12), shading="auto", cmap="magma")
plt.xlabel("time (s)")
plt.ylabel("frequency (Hz)")
plt.colorbar(label="power (dB, relative)")
plt.tight_layout()
plt.show()
```

---

## Task 1 — A synthetic signal, three tones

**Build your own script.** Create `LAB04.py` and copy each block into it, in
order. Where you see `____`, fill in the blank.

```python
FS = 20000
DURATION = 1.5
F1, F2, F3 = 300, 1500, 5000   # Hz

t = np.arange(int(DURATION * FS)) / FS
x = np.sin(2 * np.pi * F1 * t) + np.sin(2 * np.pi * F2 * t) + np.sin(2 * np.pi * F3 * t)

plt.figure(figsize=(10, 3))
plt.plot(t[:500], x[:500])     # first 500 samples only — otherwise it's a solid blob
plt.xlabel("time (s)")
plt.ylabel("amplitude")
plt.title("Three tones, mixed")
plt.tight_layout()
plt.show()

f, mag = spectrum_of(____, ____)

plt.figure(figsize=(10, 4))
plt.plot(f, mag)
plt.xlabel("frequency (Hz)")
plt.ylabel("magnitude")
plt.title("Spectrum of the mixed signal — before any filter")
plt.tight_layout()
plt.show()
```

> 📝 **Q1** Include the spectrum. You should see three peaks. Do their
> positions match `F1`, `F2`, `F3`?

---

## Task 2 — Three filters, three isolations

One tone is low, one is in the middle, one is high. Use the filter shape
that fits each: low-pass for the lowest tone, band-pass for the middle one,
high-pass for the highest.

```python
y_lp = lp_filter(x, ____,         FS)  # keeps F1
y_bp = bp_filter(x, [____, ____], FS)  # keeps F2
y_hp = hp_filter(x, ____,         FS)  # keeps F3

names = ["low-pass", "band-pass", "high-pass"]
filtered_data = [y_lp, y_bp, y_hp]

fig, axes = plt.subplots(2, 3, figsize=(15, 7))
for i in range(3):
    f_y, mag_y = spectrum_of(filtered_data[i], FS)
     
    axes[0, i].plot(f, mag, color="grey", lw=0.8, label="before")
    axes[0, i].plot(f_y, mag_y, lw=1.5, label="after")
    axes[0, i].set_title(names[i])
    axes[0, i].set_xlabel("frequency (Hz)")
    axes[0, i].legend()

    axes[1, i].plot(t[:500], x[:500], color="grey", alpha=0.5, lw=1.2, label="before")
    axes[1, i].plot(t[:500], filtered_data[i][:500], lw=1.2, label="after")
    axes[1, i].set_xlabel("time (s)")
    axes[1, i].legend()

axes[0, 0].set_ylabel("magnitude")
axes[1, 0].set_ylabel("amplitude")
plt.tight_layout()
plt.show()
```

Top row: the spectrum, same as before — one peak survives, two disappear.
Bottom row: the waveform itself, original in grey underneath, filtered on top
— this is what "isolating a tone" looks like in the time domain, not just in
the spectrum.

Pick cutoffs that sit **between** the tones, not on top of one — look back
at Task 1's spectrum to place them.

> 📝 **Q2** Include this figure. Each panel should show one peak surviving
> and two gone. For the band-pass panel specifically: why does it need two
> cutoffs where low-pass and high-pass only need one?

> 📝 **Q3** Generate the same figure with different filter orders. Try order=2, order=10 and 
> order=100. Which are the main differences? Which one is more reasonable?

---

## Task 3 — The dirty dolphin whistle

Load the recording and look at its spectrogram, unfiltered:

```python
FILENAME = "02_toothed_whale_dolphin_whistle.wav"
fs, data = wavfile.read(FILENAME)
data = data.astype(float)

f, t_spec, S = spectrogram(data, fs=fs, nperseg=1024, noverlap=768)
contour = f[np.argmax(S, axis=0)]   # loudest frequency per time slice (Lab 3)

db = 10 * np.log10(S + 1e-6)
vmin, vmax = np.percentile(db, [5, 99.5])   # sensible contrast — plain auto-scale washes this out

plt.figure(figsize=(10, 4))
plt.pcolormesh(t_spec, f, db, shading="auto", cmap="magma", vmin=vmin, vmax=vmax)
plt.plot(t_spec, contour, color="cyan", lw=1.5, label="automatic contour")
plt.ylim(0, 20000)
plt.xlabel("time (s)")
plt.ylabel("frequency (Hz)")
plt.title("Unfiltered — the contour tracker follows the hum, not the whistle")
plt.legend()
plt.tight_layout()
plt.show()
```

The cyan line locks onto the hum almost the whole time, with occasional
vertical spikes — those are **clicks**: real, broadband dolphin
echolocation sounds, momentarily louder than everything else at every
frequency. They are not noise and your filter will not remove them; keep
that in mind when you judge the result below.

**Now fix it yourself:** design one filter — pick the type, the cutoff(s)
and the order — that removes the hum and keeps the whistle. Use what you
wrote down in Task 0 to decide where the cutoff should go. Reuse `lp_filter`,
`hp_filter` or `bp_filter` from the toolbox, then repeat the spectrogram +
contour plot on the filtered signal, reusing the same `vmin`/`vmax`.

Look at the plot above and find roughly *when* the whistle happens (there is
a faint curved line — that's it). Crop both the before and after plots to a
couple of seconds around that time with `plt.xlim(...)`: the clicks
elsewhere in the recording will still pull the contour away from the
whistle, so the full 4 s view will never look clean, even with a good
filter.

There is no single right answer for the filter — a few shapes and cutoffs
will work. Explain the one you picked.

> 📝 **Q4** Include both spectrograms (before and after), zoomed to the
> whistle, with the contour on each. State the filter type, cutoff(s) and
> order you chose, and why. Does the contour now follow the whistle, inside
> that window?

---

## Task 4 — Save it, listen to it

```python
cleaned = cleaned_signal.astype(np.int16)   # your filtered signal, back to the recording's dtype
wavfile.write("dolphin_cleaned.wav", fs, cleaned)
```

Open `dolphin_cleaned.wav` in Audacity (or any media player) and listen to
it next to the original.

> 📝 **Q5** Does the cleaned recording sound different from the original?
> Describe what changed, in your own words..

---
