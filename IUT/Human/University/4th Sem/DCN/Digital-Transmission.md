# Chapter 4 — Digital Transmission

Forouzan, *Data Communications and Networking*. Covers §4.1 (Digital-to-Digital Conversion) and §4.2 (Analog-to-Digital Conversion), plus the conceptual Review Questions that test them. **Excluded per request:** the dedicated "Bandwidth" discussions/formulas (in both §4.1 and §4.2), all "Exercises" (the chapter's numeric/calculation problem set), and §4.3 Transmission Modes (parallel/serial, async/sync/isochronous) in its entirety — including the two Review Questions (11–12) that test it.

---

## 4.1 Digital-to-Digital Conversion

Converting **digital data → digital signal**. Three techniques: **line coding** (always needed), **block coding** and **scrambling** (used as needed).

### Line Coding — Core Concepts

Line coding is the sender-side encoding of a bit sequence into a digital signal (and the matching receiver-side decoding back into bits).

**Data element vs. signal element.** A **data element** is the smallest unit of information we actually want to send — a bit. A **signal element** is the shortest (timewise) unit of the actual transmitted signal — what we *can* send; data elements are the cargo, signal elements are the carrier. The ratio **r** = number of data elements carried per signal element:
- *r = 1*: one data element per signal element (simplest case).
- *r = 1/2*: it takes **two** signal elements to carry one data element (extra signal element spent purely on guaranteeing synchronization).
- *r = 2*: one signal element carries **two** data elements (more efficient use of the channel).
- *r = 4/3*: a group of 4 data elements carried by a group of 3 signal elements.

Analogy: a data element is a person needing transport; a signal element is a vehicle. *r=1* → one person per car; *r>1* → carpooling (several people per vehicle); *r<1* → a car towing an extra trailer just for one passenger's luggage (overhead).

**Data rate vs. signal rate.** **Data rate** (a.k.a. **bit rate**) = data elements (bits) sent per second, in **bps**. **Signal rate** (a.k.a. **pulse rate**, **modulation rate**, or **baud rate**) = signal elements sent per second, in **baud**. Goal of good encoding design: **maximize data rate while minimizing signal rate** — more data carried per "vehicle" sent, so less of the (expensive, limited) channel capacity is spent just moving carriers around.

**Baseline wandering.** A receiver decodes bits by comparing the incoming signal against a running-average power level it computes, called the **baseline**. A long run of the *same* bit value (all 0s or all 1s) can drag this running average off-course ("wander"), making correct decoding harder. A good line code should prevent long constant runs.

**DC components.** A signal held at a constant voltage for a while produces very-low-frequency (near-zero, "DC") spectral content. Some transmission systems (e.g. telephone lines, which can't pass frequencies below ~200 Hz; long links using transformer-based electrical isolation) **cannot carry DC components at all** — a scheme intended for such systems needs to avoid producing them.

**Self-synchronization.** The receiver's notion of "where one bit interval ends and the next begins" must track the sender's, or bits get mis-read entirely (a receiver running even slightly fast/slow will progressively drift and misinterpret an increasingly long run of bits — e.g. sender sends `10110001`, a desynchronized receiver reads `110111000011`). A **self-synchronizing** signal embeds its own timing information via built-in **transitions** the receiver can use to continuously re-lock its clock, rather than relying purely on both ends' clocks staying perfectly matched.

**Built-in error detection & noise immunity.** Some line codes have redundancy structured so that certain received patterns are simply *impossible* under the code's rules — such patterns' appearance signals a transmission error. Some are also inherently more robust against noise.

**Complexity.** More voltage levels / more elaborate rules = harder (costlier) to implement than a simple two-level scheme.

### Line Coding Schemes

Five broad families: **Unipolar**, **Polar**, **Bipolar**, **Multilevel**, **Multitransition**.

**Unipolar NRZ.** All signal levels sit on *one side* of the time axis (e.g. positive voltage = 1, zero voltage = 0). "**NRZ**" (non-return-to-zero) = the signal does not dip back to zero mid-bit. Simple, but costly (needs double the normalized power of its polar counterpart) — essentially unused today.

**Polar schemes** — voltage levels on *both* sides of the axis (e.g. positive = one bit value, negative = the other):
- **NRZ-L** ("Level"): the voltage *level itself* directly encodes the bit value.
- **NRZ-I** ("Invert"): it's the **presence/absence of a transition** (voltage change) at the start of a bit interval that encodes the value — a transition = 1, no transition = 0 (or vice versa, by convention).
 Comparing them: baseline wandering and desync are problems for *both*, but **twice as severe in NRZ-L** — a long run of either 0s or 1s skews NRZ-L, while NRZ-I only has trouble with a long run of **0s** specifically (a run of 1s in NRZ-I actually keeps producing transitions, which helps synchronization). NRZ-L also breaks completely if wiring polarity accidentally gets flipped (every 0 reads as 1 and vice versa); NRZ-I is immune to that particular failure mode, since it only cares about *change*, not absolute level. Both have a **DC component problem** — most of their energy concentrates near zero frequency.

**Polar RZ (Return-to-Zero).** Uses *three* values (positive, negative, **and zero**) — the signal always returns to 0 partway through *every* bit, giving a guaranteed mid-bit transition for synchronization regardless of data pattern. Fixes NRZ's sync weakness, but needs **twice the signal changes per bit** (more bandwidth), still suffers the "polarity flip" vulnerability, and needs 3 voltage levels (more complex to generate/discern) — essentially obsolete, superseded by biphase schemes.

**Biphase: Manchester & Differential Manchester.** Combine RZ's "transition mid-bit" idea with NRZ's two-level simplicity — the value is at one level for the first half of the bit and the other level for the second half, so there's **always** a transition exactly at the bit's midpoint, which the receiver uses purely for clock sync.
- **Manchester** = RZ-style transition timing + **NRZ-L-style** value encoding (the *direction* of the mid-bit transition, or equivalently the level during the first half, directly indicates the bit value).
- **Differential Manchester** = RZ-style transition timing + **NRZ-I-style** value encoding (the mid-bit transition is *only* for sync; the actual bit value is determined by whether there's an *additional* transition at the very start of the bit interval — presence = one value, absence = the other).

Both eliminate baseline wandering and DC components entirely (every bit contributes equal positive and negative voltage) — the price is that their signal rate is **double** that of plain NRZ, since there's always at least one transition (sync) per bit and possibly a second (value).

**Bipolar (multilevel binary): AMI & Pseudoternary.** Three voltage levels — **positive, negative, and zero** — but unlike RZ, zero-voltage represents an actual **data value**, not a mid-bit timing pulse.
- **AMI** ("Alternate Mark Inversion" — "mark" = telegraphy-speak for 1): binary 0 = neutral zero voltage; binary 1 = alternating positive/negative pulses (each successive 1 flips polarity from the last one).
- **Pseudoternary**: the mirror image — 1 = zero voltage, 0 = alternating polarity pulses.

No DC component (a long run of 1s keeps alternating polarity — no sustained constant voltage; a long run of 0s sits at a *constant* voltage of exactly zero, which by definition also carries no DC energy) — but **AMI still has a synchronization problem for long runs of 0s** (no transitions occur during them at all). This is exactly the gap that **scrambling** (below) is designed to close, letting AMI be used for long-distance links (narrow bandwidth, no DC issue) without breaking sync.

**Multilevel schemes (mBnL codes).** General idea: encode a group of *m* data bits into a group of *n* signal elements drawn from *L* possible levels, aiming to pack more bits per signal element (more bits per baud). Naming convention `mBnL`: *m* = input bit-group size, "B" = binary data, *n* = output signal-element-group size, and the second letter names *L* (B=2 levels, T=3/"ternary", Q=4/"quaternary"). Feasibility requires `2^m ≤ L^n` (enough distinct signal patterns to represent every possible data pattern); when `2^m < L^n` there's slack left over, which a well-designed code spends on synchronization guarantees, DC balance, and/or error detection.
- **2B1Q**: 2 data bits → one signal element from 4 voltage levels. No redundancy at all (2²=4¹ exactly) — used in DSL to double effective throughput vs. NRZ-L, at the cost of needing to discriminate 4 distinct voltage thresholds instead of 2.
- **8B6T**: 8 data bits → 6 ternary (3-level) signal elements (256 data patterns mapped into a subset of the 3⁶=729-ish possible signal patterns) — the large leftover redundancy is spent on synchronization, error detection, *and* active DC balancing (the sender tracks a running "weight" and inverts a whole 6-symbol group if needed to cancel out an accumulating DC bias). Used in 100BASE-T4 cable.
- **4D-PAM5**: splits data across **4 wires simultaneously**, using 5 voltage levels per wire (one level reserved purely for forward error detection). Lets Gigabit Ethernet push 1 Gbps total using 4 links that individually only need to run at a comparatively modest signal rate each.

**Multitransition: MLT-3.** Three voltage levels (+V, 0, −V) with a state-machine-style transition rule (not simply "map bit to level"): (1) bit 0 → no transition; (2) bit 1 while currently *not* at 0 → move to 0; (3) bit 1 while currently *at* 0 → move to the level **opposite** the last nonzero level visited. Net effect: a run of all-1s cycles `+V → 0 → −V → 0 → +V → …`, a repeating 4-bit-long pattern — so the *worst-case* signal frequency ends up only **one-quarter** of the raw bit rate, letting MLT-3 push a high bit rate over copper media that can't tolerate high-frequency emissions.

### Block Coding

Purpose: inject controlled **redundancy** to buy synchronization guarantees and inherent error-detection — improving on what raw line coding alone provides. General notation **mB/nB** (note the slash — distinguishes it from multilevel `mBnL` notation): a group of *m* bits is replaced by a *longer* group of *n* bits (*n > m*), via three steps — **division** (chop the stream into *m*-bit groups), **substitution** (swap each *m*-bit group for its designated *n*-bit code), **combination** (concatenate the *n*-bit groups back into one stream).

**4B/5B** — designed as a companion to NRZ-I, to fix NRZ-I's weakness with long runs of 0s. Each 5-bit output code is chosen so it has **at most one leading zero and at most two trailing zeros** — guaranteeing that however the 5-bit groups get concatenated, the combined stream never contains more than **3 consecutive 0s**. Since a 4-bit input has only 16 possible values but a 5-bit output field has 32, there are 16 "spare" codes — some assigned to control symbols (idle, halt, start/end delimiters, etc.), the rest simply left **unused**, which doubles as error detection: an arriving 5-bit group that isn't a valid assigned code signals a transmission error. Cost: signal rate goes up 20% (5 bits sent for every 4 data bits) — still much less overhead than biphase's full 2× — but 4B/5B **does not** solve NRZ-I's separate DC-component problem (if that matters, you'd need to combine it with a DC-balanced/bipolar line code instead).

**8B/10B** — same idea, scaled up: 8 data bits → 10-bit code, built internally as a combination of a 5B/6B encoder (for the top 5 bits) and a 3B/4B encoder (for the bottom 3 bits), simplifying the mapping table design. Adds a **disparity controller** that tracks the running excess of 0s-over-1s (or vice versa) and complements an entire code group when needed to keep the stream balanced (avoiding long runs in *either* direction, not just runs of 0s). With `2^10 − 2^8 = 768` unused/redundant code combinations available, it offers **better error-detection and synchronization guarantees than 4B/5B**.

### Scrambling

Motivation: biphase schemes sync well but need too much bandwidth for long-distance links; block coding + NRZ avoids that bandwidth cost but keeps NRZ's DC problem; **bipolar AMI** has narrow bandwidth *and* no DC component — its only remaining weakness is losing sync over long runs of 0s. **Scrambling** fixes exactly that weak point, *without* adding any extra bits (unlike block coding) — it substitutes specific **in-line voltage patterns** for what would otherwise be a long run of zero-level pulses, applied as part of the encoding process itself (not as a separate up-front step).

**B8ZS** (Bipolar with 8-Zero Substitution, common in North America): any run of **8 consecutive zeros** gets replaced with the pattern `000VB0VB`, where **B** = a normal-polarity bipolar pulse (following the ordinary AMI alternation rule) and **V** = a deliberate **violation** pulse (same polarity as the previous nonzero pulse — breaking the AMI alternation rule on purpose, as a recognizable signal to the receiver). The exact substitution pattern used depends on whether the previous nonzero level was positive or negative, so that the two inserted pulse-pairs always net out to two positives and two negatives — preserving DC balance while still not changing the overall bit rate.

**HDB3** (High-Density Bipolar 3-Zero, common outside North America): more conservative — substitutes after only **4** consecutive zeros (not 8), using either `000V` or `B00V`, chosen by a simple parity-style rule so the *total* count of nonzero pulses stays even after every substitution:
1. If the nonzero-pulse count since the last substitution is currently **odd** → use `000V` (adding the one V pulse makes the running count even again).
2. If it's currently **even** → use `B00V` (adding both a B and a V — two pulses — keeps the count even).

### Analog-to-Digital Conversion Recap Note
The techniques of §4.1 convert *digital data* into a *digital signal*. §4.2 (next) instead starts from an *analog* signal (e.g. from a microphone) and converts it into *digital data* — after which, §4.1's line-coding techniques can be applied on top to actually put that digital data onto a wire.

---

## 4.2 Analog-to-Digital Conversion

Two techniques: **Pulse Code Modulation (PCM)** and **Delta Modulation (DM)**.

### Pulse Code Modulation (PCM)

A PCM encoder performs three steps: **sampling → quantizing → encoding**.

**Sampling.** The analog signal is measured every *Tₛ* seconds (the **sample interval**); the **sampling rate** *fₛ = 1/Tₛ*. Three sampling methods: **ideal** (instantaneous point samples — not physically realizable), **natural** (a fast switch briefly samples the actual analog shape during each tiny window), and **flat-top** (the practical "sample-and-hold" approach — each sample is held constant until the next one). This whole sampling process is also called **PAM (Pulse Amplitude Modulation)** — but note the *result* of sampling alone is still fundamentally an analog signal (its amplitude values aren't yet restricted to a finite set) — quantization (next) is what actually digitizes it.

**Nyquist sampling theorem.** To be able to reconstruct the original signal, the sampling rate must be **at least twice the highest frequency present** in the signal. Two important nuances: (1) only a **band-limited** signal can be sampled at all (a signal with genuinely infinite bandwidth can't be); (2) the required rate depends on the **highest frequency**, not the bandwidth as such — for a **low-pass** signal these coincide (bandwidth spans 0 up to *f_max*), but for a **band-pass** signal, the bandwidth (the *width* of the occupied band, which might start well above 0 Hz) is *narrower* than *f_max* itself, so knowing only the bandwidth (without knowing exactly where that band sits) isn't enough to determine the minimum rate.
- Sampling at exactly the Nyquist rate faithfully reconstructs the signal (verified by the classic sine-wave/clock-hand thought experiments in the book); **oversampling** (above Nyquist) also works but wastes samples; **undersampling** (below Nyquist) produces **aliasing** — the reconstructed signal looks like an entirely different (usually lower-frequency, sometimes even reversed-direction) signal from the true original. (This is exactly why a forward-spinning wheel can *appear* to spin backward in a film — the camera's frame rate is undersampling the wheel's true rotation rate.)
- Real-world anchor point: telephony assumes speech tops out around 4000 Hz, so voice is sampled at **8000 samples/second**.

**Quantization.** Sampling alone still leaves *analog* (continuously-valued) sample amplitudes — quantization is the actual "make it digital" step:
1. Assume the signal's instantaneous amplitude ranges between *V_min* and *V_max*.
2. Divide that range into **L** zones, each of height **Δ = (V_max − V_min)/L**.
3. Assign each zone an integer quantization code, 0 through L−1, centered on that zone's **midpoint**.
4. Approximate each actual sample's amplitude by the code of whichever zone it falls in.

**Quantization levels.** Choice of *L* depends on how much the signal's amplitude actually varies and how accurately it must be reconstructed — a signal that barely fluctuates needs few levels; something like voice or video (constantly and widely varying) needs many (audio digitizing conventionally uses *L*=256; video, thousands). Too few levels ⇒ larger quantization error whenever there's a lot of fluctuation.

**Quantization error.** Since each real sample amplitude gets rounded to its zone's *midpoint*, there's necessarily some error unless the sample happened to land exactly on that midpoint — and this error is always bounded: **−Δ/2 ≤ error ≤ +Δ/2**. This error directly degrades the signal-to-noise ratio (and, via Shannon's theorem, the achievable channel capacity); the SNR contribution from quantization depends on the number of bits per sample *n_b*: **SNR(dB) = 6.02 n_b + 1.76**. (More levels/bits ⇒ smaller Δ ⇒ smaller worst-case error ⇒ better SNR.)

**Uniform vs. nonuniform quantization.** Many real signals' amplitudes aren't uniformly distributed (small fluctuations are far more common than large swings) — for these, **nonuniform** zone spacing (finer near low amplitudes, coarser near high amplitudes) reconstructs more accurately for the same total number of levels. This is achieved via **companding** (compressing large amplitudes down before quantizing) and **expanding** (the inverse, undone at the receiver) — giving proportionally more "quantization resolution" to weaker signal portions and less to strong ones, which measurably improves the effective quantization SNR compared to uniform spacing.

**Encoding.** Once each sample has been assigned one of *L* quantization codes, each code is simply written out as an **n_b-bit codeword**, where **n_b = log₂L**. The overall PCM output bit rate is then:

**Bit rate = sampling rate × bits per sample = fₛ × n_b**

*(Worked example from the book: digitizing voice — bandwidth up to 4000 Hz ⇒ sampling rate 8000/s; at 8 bits/sample ⇒ bit rate = 8000 × 8 = 64,000 bps = 64 kbps, the classic PCM telephony rate.)*

**Original signal recovery.** The **PCM decoder** turns each received codeword back into a held-amplitude pulse, reconstructing a staircase-shaped signal, which is then smoothed by a **low-pass filter** (matching the original signal's cutoff frequency) to recover something close to the original analog waveform — faithful reconstruction requires that the original sampling met the Nyquist condition *and* used sufficiently many quantization levels.

### Delta Modulation (DM)

Motivation: PCM is comparatively complex (full sample-quantize-encode-per-sample pipeline). **Delta modulation** simplifies drastically: instead of encoding each sample's absolute *amplitude*, it encodes only the **change (delta, δ) from the previous sample** — a **single bit per sample**, no codewords at all.

**Modulator.** The sender builds an internal reference "staircase" signal that tracks the input. At each sampling instant, it compares the actual analog input against the current staircase level: if the input is higher, it outputs bit **1** and steps the staircase **up** by δ; if lower, it outputs bit **0** and steps the staircase **down** by δ. (A delay unit holds each staircase step steady between comparisons.)

**Demodulator.** Mirrors the same staircase-building logic from the incoming bitstream (step up on 1, down on 0), then smooths the result through a **low-pass filter** to recover an approximated analog signal.

**Adaptive DM.** A refinement where the step size δ isn't fixed — it adapts to the local amplitude/slope of the analog signal, improving tracking accuracy (e.g. bigger steps when the signal is changing quickly, finer steps when it's nearly flat).

**Quantization error in DM.** DM is not error-free either — quantization error still occurs (e.g. the staircase can't perfectly track a very fast change since it's limited to one fixed step δ per sample, a limitation called "slope overload") — but in practice this error is **substantially smaller than PCM's**, at the cost of a cruder (1-bit-per-sample) representation overall.

---

## Review Questions (conceptual — excludes Transmission Mode questions 11–12, and all numeric Exercises)

**1. List three techniques of digital-to-digital conversion.**
**Line coding**, **block coding**, and **scrambling**. Line coding is always required (it's what actually turns bits into a physical signal); block coding and scrambling are optional additions used to buy better synchronization, error-detection, or DC-balance properties on top of the base line code.

**2. Distinguish between a signal element and a data element.**
A **data element** is the smallest unit of *information* we want to transmit — a bit. A **signal element** is the smallest (timewise) unit of the actual *transmitted signal* — the physical "carrier" used to move data elements across the link. Data elements are what we need to send; signal elements are what we're able to send; the ratio *r* (data elements per signal element) captures how efficiently a scheme packs the former into the latter.

**3. Distinguish between data rate and signal rate.**
**Data rate** (bit rate) = data elements (bits) transmitted per second, measured in **bps**. **Signal rate** (baud rate / pulse rate / modulation rate) = signal elements transmitted per second, measured in **baud**. A central design goal is to push the data rate as high as possible while keeping the signal rate (and hence bandwidth demand) as low as possible — i.e., carry as many bits as possible per "vehicle" sent.

**4. Define baseline wandering and its effect on digital transmission.**
Baseline wandering is the drift of a receiver's running-average power estimate (its **baseline**, used as the reference threshold for deciding bit values) caused by a long run of identical bit values (all 0s or all 1s) skewing that average away from its correct center. Effect: it makes correctly discerning subsequent bit values harder/error-prone, since the receiver's decision threshold is no longer where it should be — a good line-coding scheme should be designed to avoid producing long constant runs, precisely to prevent this.

**5. Define a DC component and its effect on digital transmission.**
A DC component is very-low-frequency (near-zero-Hz) spectral energy that arises whenever a signal's voltage stays constant for a stretch of time. Effect: some transmission media/equipment **cannot carry near-zero frequencies at all** — e.g. a telephone line can't pass anything below ~200 Hz, and long links using transformer-based electrical isolation are similarly DC-blind — so a signal with significant DC content simply cannot be transmitted correctly (or at all) over such systems, making "no DC component" a required property for line codes intended for those links.

**6. Define the characteristics of a self-synchronizing signal.**
A self-synchronizing signal carries its own timing information embedded directly in the transmitted waveform — specifically, via **transitions** placed (guaranteed, by the coding rule) at predictable points within each bit interval (start, middle, or end). These transitions let the receiver continuously reset/re-lock its clock against the incoming signal, so that even if the receiver's own clock has drifted slightly out of sync with the sender's, it gets repeatedly re-anchored and doesn't accumulate misalignment over a long bit stream.

**7. List five line coding schemes discussed in this book.**
Representative set spanning the five families: **NRZ-L** (polar), **NRZ-I** (polar), **Manchester** (polar/biphase), **AMI** (bipolar), and **2B1Q** (multilevel). *(Others covered: unipolar NRZ, polar RZ, differential Manchester, pseudoternary, 8B6T, 4D-PAM5, and MLT-3 (multitransition).)*

**8. Define block coding and give its purpose.**
Block coding (**mB/nB**) is a technique that replaces every fixed-size group of *m* data bits with a slightly *longer* group of *n* bits (*n > m*), chosen from a carefully designed mapping table. Purpose: to inject controlled **redundancy** into the bit stream *before* it's line-coded, in order to (a) guarantee synchronization properties the base line code alone can't provide (e.g. bounding the maximum run length of any single value), and (b) provide a degree of **built-in error detection** for free, since only a designed subset of all possible *n*-bit patterns are ever valid — an invalid pattern arriving signals a transmission error.

**9. Define scrambling and give its purpose.**
Scrambling is a technique, applied *as part of* (not before, unlike block coding) the line-encoding process itself, that substitutes a specific, recognizable in-line voltage pattern for what would otherwise be a long run of zero-level pulses. Purpose: to solve the synchronization-loss problem that bipolar schemes like AMI suffer from during long runs of 0s, **without increasing the transmitted bit count at all** (unlike block coding, which does add extra bits) — letting a scheme that's otherwise ideal for long-distance transmission (narrow bandwidth, no DC component) also stay reliably self-synchronizing.

**10. Compare and contrast PCM and DM.**
Both convert an analog signal into digital data via sampling. **PCM** measures and encodes the *absolute amplitude* of each sample (via sampling → quantizing into one of *L* levels → encoding as an *n_b*-bit codeword), producing a multi-bit codeword per sample; it's more accurate but more complex, and its bit rate is *fₛ × n_b*. **DM** instead encodes only the *direction of change* (up or down by a fixed step δ) relative to an internally-tracked reference "staircase," producing just a **single bit per sample** with no codewords at all — much simpler to implement, at the cost of being a cruder approximation; however, in practice its quantization error is typically **smaller** than PCM's despite (or rather, partly *because of*) that simplicity, since it's always just tracking a local, small step rather than digitizing an absolute value into coarse global zones.

---

*Review Questions 11–12 (parallel vs. serial transmission; asynchronous/synchronous/isochronous transmission) and all Exercises (13–32, numeric bandwidth/data-rate/SNR/waveform-drawing calculations) are omitted per the requested scope (Transmission Mode topic and mathematical problems excluded).*
