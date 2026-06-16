# Physical Layer

Topics related to the physical transmission of data.

- [[Multiplexing]] - Frequency Division Multiplexing (FDM), Time Division Multiplexing (TDM).

## Physical Media

Physical media are the actual physical materials used to carry signals between a transmitter and a receiver. They are categorized into two main types:

### 1. Guided Media
Signals are guided along a solid medium.

- **Twisted-Pair Copper Wire:**
    - **Description:** Two insulated copper wires, each about 1 mm thick, twisted together in a helical form to reduce electrical interference.
    - **Usage:** LANs (Category 5/6/7) and traditional telephone lines.
- **Coaxial Cable:**
    - **Construction:** Consists of two concentric copper conductors. A stiff copper wire forms the **inner conductor**, surrounded by an insulating layer. This is wrapped in a **braided outer conductor** (which acts as a shield and ground), and finally enclosed in a protective plastic jacket.
    - **Broadband vs. Baseband:** 
        - **Baseband:** Used for a single channel (e.g., traditional Ethernet).
        - **Broadband:** Supports multiple channels using Frequency Division Multiplexing (FDM), common in cable television.
    - **Attributes:** Higher bandwidth and better noise immunity than twisted-pair over longer distances.
    - **Usage:** Cable TV, Hybrid Fiber-Coax (HFC) access networks, and older Ethernet standards (10Base2/10Base5).
- **Fiber Optics:**
    - **Construction:** A thin, flexible medium made of ultra-pure glass or plastic. It consists of a **core** (where light travels), surrounded by **cladding** (with a lower refractive index to keep light in the core via total internal reflection), and a protective outer buffer/jacket.
    - **Signaling:** Bits are represented by pulses of light. A pulse might represent a '1', and the absence of a pulse a '0'.
    - **Attributes:** 
        - **High Capacity:** Supports data rates from tens to hundreds of Gigabits per second.
        - **Low Attenuation:** Signals can travel up to 100km without needing a repeater.
        - **Immune to EMI:** Being glass, it does not conduct electricity and is immune to electromagnetic interference and tapping.
    - **Usage:** Long-haul backbone links (undersea/transcontinental), high-speed local networks, and Fiber-to-the-Home (FTTH).

### 2. Unguided Media
Signals propagate freely through the atmosphere or space.

- **Radio Links:**
    - **Description:** Signals are carried in the electromagnetic spectrum. No physical "wire" is needed.
    - **Attributes:** Propagation depends on environmental conditions and distance.
    - **Types:**
        - **Terrestrial Microwave:** 
            - **Tech:** Uses highly directional antennas for line-of-sight transmission between two fixed points. 
            - **Usage:** Connecting buildings in a city or remote sites where laying cable is impractical.
        - **Wireless LAN (Wi-Fi):** 
            - **Tech:** Based on IEEE 802.11 standards. Uses shared radio frequency bands (e.g., 2.4 GHz, 5 GHz).
            - **Usage:** Providing wireless internet access within a short range (tens of meters), such as in homes or cafes.
        - **Wide-Area Cellular:** 
            - **Tech:** Uses a network of cell towers provided by carriers. Evolved from 3G to 4G (LTE) and 5G.
            - **Usage:** Mobile data access for smartphones and tablets over broad geographic areas.
        - **Satellite:** 
            - **GEO (Geostationary):** Satellites orbit at ~36,000 km, appearing fixed in the sky. High propagation delay (~280ms).
            - **LEO (Low Earth Orbit):** Satellites like Starlink orbit much closer (~550 km), offering much lower latency.

### Examples
- **Ethernet Cable (Cat 6):** Used to connect a PC to a router in an office (Guided).
- **Home Wi-Fi:** Signals traveling through air to your phone (Unguided).
- **Undersea Cables:** Massive bundles of fiber optics connecting continents (Guided).
