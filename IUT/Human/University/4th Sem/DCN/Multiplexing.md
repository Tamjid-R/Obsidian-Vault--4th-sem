# Multiplexing

Multiplexing is the set of techniques that allows the simultaneous transmission of multiple signals across a single data link.

## Frequency Division Multiplexing (FDM)

### Detailed Note
Frequency Division Multiplexing (FDM) is an analog multiplexing technique that combines several signals into a single medium by dividing the available bandwidth into distinct frequency bands.

- **Bandwidth Division:** The total bandwidth of the communication link is partitioned into multiple non-overlapping sub-channels.
- **Carrier Frequencies:** Each signal is modulated onto a different carrier frequency.
- **Simultaneity:** All signals are transmitted at the same time over the same physical medium.
- **Guard Bands:** To prevent signals from overlapping and causing interference (crosstalk), small strips of unused bandwidth called "guard bands" are placed between the allocated frequency bands.
- **Requirement:** The bandwidth of the link must be greater than the combined bandwidths of the signals to be transmitted.

### Feynman Explanation
Imagine a massive highway with multiple lanes. FDM is like giving each car its own dedicated lane. 
- The **highway** is the physical wire or air through which the data travels.
- The **lanes** are the different frequency bands.
- Even though all the cars are moving along the same highway at the same time, they don't crash into each other because they stay in their own lanes. If you want to "hear" a specific car (data), you just look at the specific lane it’s in.

### Examples
- **Radio Broadcasting:** This is the most common example. Each radio station is assigned a specific frequency (e.g., 98.1 MHz, 104.5 MHz). When you tune your radio, you are selecting a specific "lane" or frequency band to receive that station's signal.
- **Cable TV:** A single cable coming into your house carries hundreds of channels. Each channel is transmitted on a different frequency band within that cable.
- **First-Generation Cellular Phones:** Used FDM to assign a specific frequency band to each user for a call.

---

Q. What is frequency division multiplexing (FDM): different channels transmitted in different frequency bands?
ANS: Frequency Division Multiplexing (FDM) is an analog multiplexing technique where the total bandwidth of a shared medium is divided into a series of non-overlapping frequency sub-bands. Each sub-band is used to carry a separate signal (channel) simultaneously. To prevent interference between these channels, "guard bands" are used to separate them.

---
*See also:* [[Physical Layer]], [[Chapter 1 Summary]]
