


# ACUS Lab 1 — Files, Vectors & Logarithms

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

Welcome to your first hands-on session in ACUS. Everything you need for today is explained below as we go. By the end of this lab you'll be able to load real numbers into Python, work with them efficiently, and understand *why* underwater acoustics measures sound in decibels rather than plain pressure.

Work through the tasks in order — each one builds on the last. Where you see `____`, that's a blank for you to fill in.

### Objectives
* Python basics, including files, vectors, functions and logarithms
* Understanding the difference between pressure and decibels

### What to hand-in
Deliver to Atenea the following files: 
* A report in PDF format with snapshots of your code explaining every task.
* In some tasks there are questions, marked with 📝, that need to be answered in your report.
* The report must include an `AI usage` section describing which tools you used and why.
* The final python script containing all the tasks in a single file name `LAB1.py`

---

## Before you start: three programming ideas you'll use today

### Variables
A variable is a name that stores a value so you can reuse it without retyping it — think of it as a labelled box.

```python
depth = 25.4        # a number
sensor_name = "H1"  # some text
```
You can put a new value in the box at any time, and refer to it by name anywhere afterwards.

### Arrays (Lists in Python)
Most of the time we don't work with a single value but a whole sequence of them — e.g. one pressure reading every second from a sensor. Python's built-in way to store an ordered sequence of values is a **list**, written with square brackets:

```python
readings = [1.5, 2.1, 1.8, 3.0]
print(readings[0])    # first element -> 1.5   (Python counts from 0!)
print(readings[1])    # second element -> 2.1  
print(readings[-1])   # last element -> 3.0
print(readings[-2])   # next-to-last -> 1.8

# len() returns the number of elements
print(len(readings))  # number of elements -> 4
```

Later in this lab we'll upgrade from plain lists to **NumPy arrays** — the same basic idea, but built to do maths on every element at once instead of one at a time.

```python
import numpy as np  # import the numpy library

readings = [1.5, 2.1, 1.8, 3.0]  # a plain Python list
print("Regular list:", readings)  # simple python3 built-in list

readings_array = np.array(readings)
print("Numpy array :", readings_array)  # numpy array

# Now it's much simpler to do operations over the whole vector:
print("Max:", readings_array.max())
print("Min:", readings_array.min())
print("Mean:", readings_array.mean())
print("Sum:", readings_array.sum())


```


### Functions
A function is a named, reusable recipe: you give it some input, it does some work, and it hands back a result. This means you write a calculation once and reuse it as many times as you like.

```python
def add_five(x):
    return x + 5

print(add_five(10))   # -> 15
```

We can have as many arguments as needed in a function: 
```python
def add_numbers(x, y):
    return x + y

print(add_numbers(10, 30))   # -> 40
print(add_numbers(20, 5))    # -> 25
```


Today you'll write a small function that converts a pressure reading into decibels — once written, you can apply it to any value or any list of values.

---

# Lab Assignment

**Goal:** by the end you'll have one script (`LAB1.py`) that reads a sensor file, converts pressures to decibels, and plots them — plus a PDF report. Each task adds one piece; blanks (`____`) are for you to fill in, and 📝 marks questions to answer in the report. 

## Task 1 — Hello, ocean

**Goal:** get comfortable with variables and `print()` before anything acoustic enters the picture.  
**Report**: Add a snapshot of the code to your report.  

```python
name = "____"                     # TODO: put your name (or your pair's names) here
print("Hello, my name is:", name)

a = 12
b = 5
print("Sum:", a + b)
print("Difference:", a - b)
print("Product:", a * b)
```
> 📝 Make sure that the program prints your names as soon as the program starts

---

## Task 2 — Reading the sensor log into a list

**Goal:** get a real dataset into Python and print the number of readings, plus the first and the last ones.   


You've been given `sensor_log.txt` — a text file with one pressure reading (in µPa) per line, recorded by a toy sensor over time. Before we can do anything with it, we need to read it into a list.

```python
readings = []                       # start with an empty list
with open("sensor_log.txt", "r") as f:
    for line in f:
        value = float(line)         # turn the text on this line into a number
        readings.append(value)       # add `value` to the end of the readings list
                                     
```

Now check what you loaded:

```python
print("Number of readings:", ____(readings))     # TODO: how many items are in a list?
print("First reading:", readings[____])          # TODO: index of the first element
print("Last reading:", readings[____])           # TODO: index of the last element
```

---

## Task 3 — From lists to NumPy arrays

**Goal:** see why we upgrade from plain lists to arrays once we start doing maths.  
**Report**: Add a snapshot of the code to your report.  

With a plain list, finding an average means writing a loop:
```python
total = 0
for r in readings:
    total = total + r
average = total / len(readings)
print(average)
```

NumPy gives us **arrays**: same idea as a list, but built for fast, one-line maths across every value at once.

```python
import numpy as np

readings_arr = np.array(readings)    # convert the list into a NumPy array

print("Max:", readings_arr.____())   # TODO: NumPy method for the maximum value
print("Min:", readings_arr.____())   # TODO: NumPy method for the minimum value
print("Mean:", readings_arr.____())  # TODO: NumPy method for the average value
```

**Bonus:** `np.argmax(readings_arr)` gives you the *position* of the largest value, not the value itself. Try it — which reading (by index) was the loudest?

---

## Task 4 — Plotting your data

We won't explain how plotting works under the hood today — just how to use it. The cell below draws a line graph. **X and Y must have the same number of values, or you'll get an error.**

```python
import matplotlib.pyplot as plt

time = np.arange(len(readings_arr))   # one time-step per reading: 0, 1, 2, 3, ...

plt.plot(____, ____)     # TODO: put your X data and Y data here, in that order
                          
plt.xlabel("time (s)")
plt.ylabel("pressure (µPa)")
plt.show()
```

> 📝 **Look at your plot.** Can you clearly see the quiet readings? Or does one or two loud readings dominate the whole graph, squashing everything else near zero? Keep this in mind for the next task.

---

## Task 5 — Why does acoustics use logarithms?

Underwater sound pressures span an enormous range — from around 1 µPa (extremely faint) up to well over 100,000 µPa for a loud nearby event. That's a ratio of 100,000-to-1 or more, all in the same recording. Numbers spanning that many orders of magnitude are hard to plot, compare, or even talk about directly — as you probably just saw in Task 4.

The **decibel (dB)** is a logarithmic unit built to compress this huge range into something manageable. For pressure, the formula is:

**dB = 20·log10( p / p_ref )**

where `p_ref` is a fixed reference pressure. Underwater, the standard reference is **1 µPa**, so levels are written `dB re 1 µPa`. In air, reference pressure is 20 µPa, so air and underwater measurements are not directly comparable. 

**Example:** a pressure of 1000 µPa gives
20·log10(1000 / 1) = 20·3 = **60 dB re 1 µPa**

A pressure exactly equal to the reference (1 µPa) is always **0 dB**, by definition.

### Write the conversion function

```python
def pressure_to_db(p, p_ref=1.0):
    return ____ * np.log10(____ / ____)   # TODO: complete the formula above (20, p, p_ref)

print(pressure_to_db(1.0))       # should print 0.0
print(pressure_to_db(1000.0))    # should print 60.0
```

### Apply it to your whole recording

```python
db_values = pressure_to_db(____)     # TODO: apply the function to your whole readings_arr

fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(10, 4))
ax1.plot(time, readings_arr)
ax1.set_title("Linear")
ax1.set_xlabel("time (s)")
ax1.set_ylabel("pressure (µPa)")

ax2.plot(____, ____)             # TODO: plot time vs. db_values here
ax2.set_title("Logarithmic")
ax2.set_xlabel("time (s)")
ax2.set_ylabel("pressure (dB re 1 µPa)")

plt.show()
```
> 📝 **Compare the two plots.** Which one actually lets you see the quiet readings *and* the loud ones at the same time?

---

## Task 6 — Can you just add decibels?

Here's a question to think about before you code anything: suppose one source produces 100 dB re 1 µPa, and an identical second source starts up right next to it. What's the combined level?

It is **not** 200 dB — decibels are a logarithmic scale, not a linear one, so they don't add like ordinary numbers. To combine two levels correctly, you have to: convert each one back to a linear quantity, add those linear quantities, then convert the total back to dB.

```python
import numpy as np 

def combine_db_levels(db_arr):
    linear_arr = np.power(____, db_arr / ____)  # convert all db values to linear
    linear_add = linear_arr.sum()  # sum all linear values into a single value
    return ____ * np.log10(_____)  # convert the sum back to dB

db_array = np.array([100, 100])  # Create a numpy array with dB values
print("combined:", combine_db_levels(db_array))
```

> ⚠️ Warning: This exercise is only meant to illustrate that **dB values cannot simply be added**. Here, we convert the dB levels back to acoustic pressure, add the pressures, and then convert the result back to dB. This implicitly assumes that the sound sources are **coherent and arrive in phase**. In general, acoustic pressures cannot be added this way unless this condition is satisfied.

---
## Wrap-up

Today you learned how to:
- store and reuse values with **variables**
- read a real data file into a **list**
- upgrade to **NumPy arrays** for fast maths across a whole dataset
- plot data by supplying matching X and Y arrays
- convert pressure to **decibels**, and why underwater acoustics needs that logarithmic scale
- correctly **combine** two sound levels expressed in dB

