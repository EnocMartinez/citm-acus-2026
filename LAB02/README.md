# ACUS Lab 2 — Recording the Sea with a HydroMoth

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

In Lab 1 you worked with numbers somebody else had put in a text file. Today you are going to
**make your own data**: you will configure a real hydrophone, record real underwater sound, pull
the file off it, and open it in Python.

The session has two halves:

* **Part A — the device.** Configure a HydroMoth, record, and recover the file. Hands on hardware.
* **Part B — the data.** Open your own recording in Python, find out what is inside it, and plot
  it against real clock time.

Everything you need is explained as we go. Where you see `____`, that is a blank for you to fill
in, and 📝 marks a question to answer in your report.

> **If you have already had theory Session 3**, Part B will feel familiar — you have seen WAV
> headers and sampling rates on slides. Here you do it on your own file instead of a supplied one.
> **If you have not had Session 3 yet**, don't worry: everything you need is explained below, and
> the theory session will fill in the background afterwards. Neither order puts you behind.

### Objectives

* Configure a low-cost hydrophone and make a real recording
* Understand what a WAV file contains and how to read its parameters in Python
* Turn a sample index into an actual date and time
* Convert raw ADC counts into pressure and plot it

### What to hand in

Deliver to Atenea the following files:

* A report in PDF format with snapshots of your code explaining every task.
* The answers to every question marked with 📝.
* **Your own recording** (one WAV file). If your group's device failed, say so and use the
  supplied one.
* The report must include an `AI usage` section describing which tools you used and why.
* The final Python script containing all the tasks in a single file named `LAB02.py`.

---

# Part A — The HydroMoth

## What it is

A **HydroMoth** is an [AudioMoth](https://www.openacousticdevices.info/) board inside a waterproof
housing. It is a *low-cost* recorder: it costs roughly one or two hundred euros, where a research
hydrophone costs several thousand. The trade-off is that it is noisier and less precisely
calibrated — but it is cheap enough that you can put several in the water, and good enough for a
great deal of real science.

What you need to know to use one:

* It records to a **microSD card**, as **16-bit mono WAV** files.
* It has **no clock battery you can rely on** and **no screen**. It knows the time only because
  the configuration app told it, and it reports everything in **UTC**.
* It records on a **schedule** you set in advance. Once it is sealed, it does exactly what it was
  told and nothing else. Today the devices will be set to **record continuously**, so each device
  produces **one single WAV file** covering the whole deployment.
* The file is named after the time the recording started: `YYYYMMDD_HHMMSS.WAV`.
  So `20261015_103000.WAV` started at **10:30:00 UTC on 15 October 2026**.

That last point matters more than it looks. The filename is the *only* thing tying your data to
the real world, and you will use it in Part B.

## Task A1 — Configure the device

Work in pairs. One laptop, one device.

1. Install the **AudioMoth Configuration App** from
   [openacousticdevices.info](https://www.openacousticdevices.info/applications).
2. Open the device, insert the microSD card, set the switch to **USB/OFF**, and connect it by USB.
   The app should show the device ID and the battery voltage.
3. Check the **time**. The app sets the device clock from your computer, in UTC. Press the button
   that sets the time and confirm it took.
4. Set the **sampling rate** and the **gain**. Your instructor will tell you which values to use
   for today's session.
5. Set the **recording schedule to continuous**. Every device records non-stop for the whole
   deployment, so there is no duty cycle to configure.
6. Write the configuration to the device.

> 📝 Write down, for your device: its ID, the sampling rate, the gain setting, and the
> battery voltage the app reported. You will need these in your report.

> 📝 The app sets the clock in **UTC**, but your laptop shows local time. In October,
> Barcelona is UTC+2. If you start a recording at 11:30 on your watch, what time will appear in
> the filename? Why do you think recorders work in UTC rather than local time?

## Task A2 — Seal it and record

1. Set the switch to the position your instructor tells you, before closing the housing. A
   device left on USB/OFF records nothing — this is the single most common mistake.
2. Close the housing carefully. Check the **O-ring**: it must be clean, seated in its groove, and
   lightly greased. A hair or a grain of sand on the O-ring floods the device.
3. Deploy it as instructed
4. **Make some noise.** Tap the side of the tank, talk near the water, splash, scrape two stones
   together. Note roughly *when* you did each thing: you want to be able to find your own events
   in the data later.

## Task A3 — Recover the data

1. Recover the device, dry the **outside** thoroughly before opening it. Water on the lid runs
   straight into the electronics otherwise.
2. Open the housing, take out the microSD card, and copy the WAV file to your laptop.
3. Put it in the `data/` folder next to your script. **Do not rename it** — the filename is
   your timestamp.

---

# Part B — Opening your recording

## Before you code: what is inside a WAV file

A WAV file has exactly two parts:

```
+---------------------------+------------------------------------------+
|          HEADER           |                 SAMPLES                  |
|      a few dozen bytes    |            everything else               |
+---------------------------+------------------------------------------+
```

The **samples** are just a long list of whole numbers — the raw output of the analog-to-digital
converter inside the recorder. We call them **counts**. They are not volts and not pascals: they
are dimensionless numbers, and for a 16-bit recorder they run from −32768 to +32767.

A list of numbers on its own is useless, so the **header** stores the three things you need to
make sense of them:

| Header field | What it tells you |
|---|---|
| **number of channels** | 1 = mono, 2 = stereo, N = an array of hydrophones |
| **sampling rate** (Hz) | how many samples were taken per second |
| **bits per sample** | how many bits each number uses — 16 for a HydroMoth |

That is why you cannot just open a WAV in a text editor: almost everything in it is numbers, not
characters. Try it once if you like — you will get a screen of garbage with the words `RIFF`,
`WAVE` and `data` scattered through it. Those are labels marking the parts of the file. The file
is not broken; a text editor is simply the wrong tool.

### Opening one in Python

We will use `scipy.io.wavfile`, which gives you the sampling rate and the samples in one go:

```python
from scipy.io import wavfile

fs, data = wavfile.read("data/20261015_103000.WAV")

print("sampling rate :", fs)          # samples per second, in Hz
print("data type     :", data.dtype)  # int16 → 16 bits per sample
print("shape         :", data.shape)  # (n_frames,) for mono, (n_frames, 2) for stereo
```

Three things to notice:

* `fs` comes straight out of the header.
* `data.dtype` tells you the bit depth: `int16` means 16 bits per sample.
* `data.shape` tells you the length **and** the number of channels. A mono file gives you
  `(1440000,)` — one number. A stereo file gives you `(1440000, 2)` — two numbers.

From those you get the duration, which is **not** stored in the header. You compute it:

```
duration = number of frames / sampling rate
```

A **frame** is one sample per channel. For mono, frames and samples are the same thing; for stereo
there are two samples per frame.

---

# Lab Assignment

**Goal:** by the end you will have one script (`LAB02.py`) that opens your own recording, reports
what is inside it, and plots pressure against real clock time.

Start your script with:

```python
import os
from datetime import datetime

import numpy as np
import matplotlib.pyplot as plt
from scipy.io import wavfile
```

## Task 1 — Find your file

Locate the WAV file in your disk and create a variable with your file name

```python
path = "MY_FILE.wav"
print("File name:", path, "size", os.path.getsize(path), "bytes")
```

---

## Task 2 — Open it and read its parameters

**Goal:** get the three header numbers, and work out the duration.

```python
fs, data = wavfile.read(path)

n_frames = len(data)
duration = ____ / ____                 # TODO: frames and sampling rate — which way round?

print("sampling rate :", fs, "Hz")
print("data type     :", data.dtype)
print("n frames      :", n_frames)
print("duration      :", duration, "s")
print("first 10 counts:", data[:10])
```

> 📝 Report the sampling rate, bit depth, number of channels and duration of your file.
> Does the duration match the length you asked the device to record?

### A quick check on size

```python
bytes_per_second = fs * ____ * ____    # TODO: bytes per sample, and number of channels
print("bytes per second :", bytes_per_second)
print("MB for this file :", bytes_per_second * duration / 1e6)
print("actual file size :", os.path.getsize(path) / 1e6, "MB")
```

> 📝 Your computed size and the real file size should agree to within a few hundred bytes.
> Where does the small difference come from?

> 📝 At this sampling rate and bit depth, how many **GB per day** would the device write if
> it recorded continuously? Would a 32 GB card last a week?

---

## Task 3 — What the device wrote in the header

**Goal:** see that the header carries more than the three numbers.

A HydroMoth writes a short text comment into the header: the device ID, the gain setting, the
battery voltage and the temperature at the moment the recording started. `scipy` ignores it, so we
go and look for it ourselves. Don't worry about how this code works — it just searches the raw
bytes of the file for readable text.

```python
with open(path, "rb") as f:            # "rb" = read as raw BYTES, not text
    header_bytes = f.read(400)         # the first 400 bytes are enough

text = "".join(chr(b) if 32 <= b < 127 else "." for b in header_bytes)
print(text)
```

> 📝 Copy the comment you found into your report. Does the device ID match the one the
> configuration app showed you in Task A1? Does the gain setting match what you configured?

> 📝 The battery voltage and the temperature are in there too. Why might those be worth
> recording alongside the audio?

---

## Task 4 — A time axis, in seconds

**Goal:** stop plotting against "sample number" and start plotting against time.

Sample number `n` was taken at time `t = n / fs` seconds after the recording started. So:

```python
t = np.arange(n_frames) / ____         # TODO: what do you divide by?

print("first times :", t[:5])
print("last time   :", t[-1], "s")

plt.figure(figsize=(12, 4))
plt.plot(____, ____)                   # TODO: time on X, counts on Y
plt.xlabel("time since start of recording (s)")
plt.ylabel("ADC counts")
plt.title(filename)
plt.show()
```

> 📝 Include the plot. Can you see any of the noises you made in Task A2? Roughly when do
> they happen?

> 📝 What is the largest count in your recording (`np.abs(data).max()`)? A 16-bit recorder
> saturates at 32767. How close did you get? What would the waveform look like if you had gone
> over?

---

## Task 5 — A time axis with the real date and time

**Goal:** put your recording on the actual clock, using the start time in the filename.

The filename is `YYYYMMDD_HHMMSS.WAV`, which Python can parse directly:

```python
stem = filename.replace(".WAV", "")         # "20261015_103000"
start = datetime.strptime(stem, "____")     # TODO: the format: "%Y%m%d_%H%M%S"
print("recording started at", start, "UTC")
```

Now add your seconds-since-start to that moment. NumPy can do the whole vector at once:

```python
t_abs = np.datetime64(start) + (1e6 * t).astype("timedelta64[us]")

print("first timestamp :", t_abs[0])
print("last timestamp  :", t_abs[-1])

plt.figure(figsize=(12, 4))
plt.plot(____, ____)                        # TODO: absolute time on X, counts on Y
plt.xlabel("time (UTC)")
plt.ylabel("ADC counts")
plt.title("HydroMoth recording — " + str(start) + " UTC")
plt.gcf().autofmt_xdate()                   # tidy up the date labels
plt.show()
```

> 📝 Include the plot. Compare the timestamps on the X axis with the log you kept in Task
> A3 — remember your log is in local time and the recorder is in UTC. Do your noises land where
> you expected?

> 📝 Suppose the configuration app had never set the clock, and the device started from
> its factory date. Everything you have done so far would still run perfectly and produce a
> beautiful plot. What exactly would be wrong with it?

---

## Task 6 — From counts to pressure

**Goal:** turn dimensionless counts into something physical, using the chain from Session 2.

The counts are what the ADC produced. To get back to the pressure at the hydrophone you undo the
chain, exactly as in the theory session:

**p[n] = x[n] · LSB / (S · G)**

where

* `x[n]` is the count,
* `LSB = FSR / 2^N` is the voltage of one count,
* `S` is the hydrophone sensitivity in V/µPa,
* `G` is the gain of the amplifier.

For our devices the amplifier gain is already included in the sensitivity figure, so `G = 1` and
you only need `S`. **Use the values your instructor gives you for the units you used today** —
calibration is specific to the device and the gain setting:

```python
# --- calibration constants for our HydroMoths  ---
N_BITS = 16  # bits
FSR = 2.5  # Volts              
Sensitivity = -165.0  # dB re 1 uPa 
Gain = 10 # dB
# ---------------------------------------------------------------------------------

lsb = FSR / 2 ** ____                      # TODO: how many levels does an N-bit ADC have?
S = 10 ** (____ / 20)                      # TODO: dB back to linear — amplitude quantity!

print("one count is", lsb * 1e6, "µV")
print("sensitivity  is", S, "V/µPa")

pressure = data # Convert ADC counts to uPa, type your formula here
```

```python
plt.figure(figsize=(12, 4))
plt.plot(t_abs, pressure / 1e6)            # µPa → Pa, so the numbers stay readable
plt.xlabel("time (UTC)")
plt.ylabel("pressure (Pa)")
plt.title("Pressure at the hydrophone")
plt.gcf().autofmt_xdate()
plt.show()
```

> 📝 What is the loudest pressure in your recording, in Pa? Convert it to a level in
> dB re 1 µPa using the function you wrote in Lab 1.

> 📝 What is the largest pressure this device could have recorded at this gain, before
> clipping? (Hint: the biggest count is 32767.) What would you change if you wanted to record
> something louder?

---


## Wrap-up

Today you learned how to:

- configure a real hydrophone, and why it keeps time in **UTC**
- recover data from a device and treat the **filename as the timestamp**
- open a WAV in Python and read its **sampling rate, bit depth and number of channels**
- compute **duration** and file size from those three numbers
- build a time axis with `t = n / fs`, and anchor it to a **real date and time**
- convert **ADC counts to pressure** with the chain from Session 2
- zoom into a window of a recording instead of plotting all of it

Next lab we move into the frequency domain and ask what your recording was actually made of.
