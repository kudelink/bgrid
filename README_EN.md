# BGrid Coordinate System

[![English](https://img.shields.io/badge/lang-en-red.svg)](README_EN.md) 
[![Español](https://img.shields.io/badge/lang-es-yellow.svg)](README.md)

![images/bgrid_general.png](images/bgrid_general.png)

## Web for testing: https://bgrid.org

## Summary

We present a new coordinate system called "BGrid" designed to reference locations on the Earth's surface through hierarchical divisions. The system recursively divides the Earth's surface into 2048 plots per level, reaching a precision of up to 29 m². A distinctive feature is the alternation in division factors between longitude and latitude to obtain more homogeneous areas, as well as the possibility of representing coordinates using BIP39 standard words, facilitating their memorization and verbal communication.

## 1. Introduction

Traditional geographic coordinate systems such as decimal coordinates (DD) or UTM coordinates have certain limitations in terms of ease of verbal communication, memorization, and variable precision. The BGrid system offers an alternative that allows:

1. Referencing geographic areas with different levels of precision according to needs.
2. Facilitating verbal and written communication of coordinates through the optional use of words.
3. Maintaining a relatively equitable distribution of the area of the plots at different latitudes.
4. Implementing a system compatible with modern encryption and information storage technologies.

## 2. Foundations of the BGrid System

### 2.1 Hierarchical Structure

The BGrid system divides the Earth's surface into 2048 initial plots (level 1), and each of these plots can be further subdivided into 2048 smaller plots (level 2), and so on until reaching level 4 of precision. This results in a potential total of 2,048⁴ plots at the most detailed level.

### 2.2 Alternating Divisions

To correct the distortion that occurs in traditional coordinate systems, where cells narrow near the poles, BGrid implements an alternation in division factors:

- In even levels: longitude is divided into 32 parts and latitude into 64 parts
- In odd levels: longitude is divided into 64 parts and latitude into 32 parts

This alternation produces plots closer to a square at different latitudes.

### 2.3 Coordinate Representation

BGrid coordinates are represented by a series of up to four numbers separated by commas, where each number corresponds to the plot identifier at its respective level:

```
N₁,N₂,N₃,N₄
```

Where:
- N₁: Level 1 plot identifier (between 1 and 2,048)
- N₂: Level 2 plot identifier (between 1 and 2,048) within N₁
- N₃: Level 3 plot identifier (between 1 and 2,048) within N₂
- N₄: Level 4 plot identifier (between 1 and 2,048) within N₃

### 2.4 Precision per Level

Considering an approximate Earth surface area of 510,000,000 km², the precision per level is:

- Level 1: 510,000,000 / 2,048 = 249,023.44 km²
- Level 2: 510,000,000 / 2,048² = 121.59 km²
- Level 3: 510,000,000 / 2,048³ = 0.059 km² (59,371 m²)
- Level 4: 510,000,000 / 2,048⁴ = 0.000029 km² (29 m²)

## 3. Mathematical Formulation

### 3.1 Encoding Process (DD Coordinates to BGrid)

To convert traditional decimal coordinates (latitude, longitude) to BGrid coordinates, the following algorithm is used:

#### 3.1.1 Coordinate Normalization

First, we normalize the input coordinates to obtain values between 0 and 1:

For longitude:
$$x = \frac{\text{lon} + 180}{360}$$

For latitude:
$$y = \frac{\text{lat} + 90}{180}$$

Where:
- lon: decimal longitude in the range [-180, 180]
- lat: decimal latitude in the range [-90, 90]
- x: normalized longitude in the range [0, 1]
- y: normalized latitude in the range [0, 1]

#### 3.1.2 Index Calculation for Each Level

To determine the plot index at each level (N₁, N₂, N₃, N₄), we iteratively apply:

For level i (where i goes from 1 to 4):

If i is odd:
$$D_{\text{lon}} = 64$$
$$D_{\text{lat}} = 32$$

If i is even:
$$D_{\text{lon}} = 32$$
$$D_{\text{lat}} = 64$$

Where $$D\_{\text{lon}}$$ and $$D\_{\text{lat}}$$ are the division factors for longitude and latitude respectively.

Then we calculate:
$$\text{col}\_i = \lfloor x \cdot D\_{\text{lon}} \rfloor$$
$$\text{fila}\_i = \lfloor y \cdot D\_{\text{lat}} \rfloor$$
$$N\_i = \text{fila}\_i \cdot D\_{\text{lon}} + \text{col}\_i$$

And we update the normalized coordinates for the next level:
$$x = \frac{x \cdot D\_{\text{lon}} - \text{col}\_i}{1}$$
$$y = \frac{y \cdot D\_{\text{lat}} - \text{fila}\_i}{1}$$

### 3.2 Decoding Process (BGrid to DD Coordinates)

To convert BGrid coordinates to decimal coordinates, we use the following algorithm:

#### 3.2.1 Initialization

We start with the full ranges of longitude and latitude:
- Longitude range: [-180, 180]
- Latitude range: [-90, 90]

#### 3.2.2 Refinement per Level

For each level i (from 1 to n, where n is the number of levels provided in the BGrid coordinate):

If i is odd:
$$D_{\text{lon}} = 64$$
$$D_{\text{lat}} = 32$$

If i is even:
$$D_{\text{lon}} = 32$$
$$D_{\text{lat}} = 64$$

We calculate the row and column from index Ni:

```math
\text{col}_i = N_i \bmod D_{\text{lon}}
```

```math
\text{fila}_i = \lfloor N_i / D_{\text{lon}} \rfloor
```

We update the coordinate ranges:

```math
\text{ancholon} = \frac{\text{maxlon} - \text{minlon}}{D_{\text{lon}}}
```

```math
\text{altolat} = \frac{\text{maxlat} - \text{minlat}}{D_{\text{lat}}}
```

```math
\text{minlon} = \text{minlon} + \text{col}_i \cdot \text{ancholon}
```

```math
\text{maxlon} = \text{minlon} + \text{ancholon}
```

```math
\text{minlat} = \text{minlat} + \text{fila}_i \cdot \text{altolat}
```

```math
\text{maxlat} = \text{minlat} + \text{altolat}
```

#### 3.2.3 Final Coordinate Calculation

Once all levels are processed, the decimal coordinates (lat, lon) of the plot center are:

```math
\text{lon} = \frac{\text{minlon} + \text{maxlon}}{2}
```

```math
\text{lat} = \frac{\text{minlat} + \text{maxlat}}{2}
```

## 4. Mnemonic Representation through BIP39

### 4.1 Foundations of BIP39

BIP39 (Bitcoin Improvement Proposal 39) defines a standard for generating mnemonic phrases from numbers using a list of 2,048 words. This standard was originally designed to represent cryptocurrency wallet keys in a recallable way.

### 4.2 Application in BGrid

Each plot index (N₁, N₂, N₃, N₄) can be converted to its corresponding word in the BIP39 list:

$$\text{word}_i = \text{BIP39}[N_i]$$

Where BIP39[n] returns the word at position n of the BIP39 list (values from 1 to 2,048).

For example, a BGrid coordinate '1045,45,123,319' could be represented as:
"llover,agoniá,apetito,calle"

### 4.3 Multilingual Advantages

The BIP39 standard is available in multiple languages, allowing the representation of BGrid coordinates in the user's preferred language. For example, the same coordinate above could be represented in English as:
"little,airport,aunt,chief"

## 5. Comparative Analysis with Other Systems

### 5.1 Comparison with Decimal Coordinates (DD)

| Feature | BGrid | Decimal Coordinates |
|----------------|---------|------------------------|
| Maximum precision | 29 m² | Variable according to decimals |
| Ease of verbal communication | High (using BIP39 words) | Low |
| Hierarchical structure | Yes | No |
| Precision levels | 4 defined levels | Undefined (depends on decimals) |
| Distortion at high latitudes | Partially corrected | High |

### 5.2 Comparison with Other Hierarchical Systems

| Feature | BGrid | Geohash | What3Words |
|----------------|---------|---------|------------|
| Numerical base | 2,048 | 32 | N/A |
| Maximum precision | 29 m² | Variable | 9 m² |
| System openness | Open | Open | Proprietary |
| Compatibility with cryptography | High | Medium | Low |
| Area distribution | Adaptive | Fixed | Fixed |

## 6. Practical Applications

### 6.1 Geolocation with Variable Privacy

The BGrid system allows sharing locations with different levels of precision according to privacy requirements:

- Level 1: Useful for indicating wide regions (approximately 250,000 km²)
- Level 2: Suitable for metropolitan areas or regions (122 km²)
- Level 3: Sufficient precision to locate neighborhoods or specific areas (59,371 m²)
- Level 4: Precision to locate specific buildings (29 m²)

### 6.2 Integration with Blockchain Systems

Compatibility with BIP39 facilitates the integration of BGrid into cryptographic applications for property registration, location certification, or proofs of location.

### 6.3 Verbal Communication of Coordinates

The ability to represent coordinates as words facilitates:
- Phone dictation of locations
- Memorization of important places
- Reduction of errors in the verbal transmission of locations

## 7. Implementation

### 7.1 Pseudocode Algorithm for Encoding

```
function coordinatesToBGrid(latitude, longitude, levels):
    // Coordinate normalization
    x = (longitude + 180) / 360
    y = (latitude + 90) / 180
    
    result = []
    
    for i from 1 to levels:
        if i is odd:
            divisorLon = 64
            divisorLat = 32
        else:
            divisorLon = 32
            divisorLat = 64
        
        column = integer(x * divisorLon)
        row = integer(y * divisorLat)
        
        // Index calculation
        index = row * divisorLon + column
        add index to result
        
        // Update for next level
        x = (x * divisorLon - column)
        y = (y * divisorLat - row)
    
    return result joined by commas
```

### 7.2 Pseudocode Algorithm for Decoding

```
function bgridToCoordinates(coordBGrid):
    // Separate levels
    levels = separateByCommas(coordBGrid)
    
    // Initialize ranges
    minLon = -180
    maxLon = 180
    minLat = -90
    maxLat = 90
    
    for i from 1 to length(levels):
        index = levels[i]
        
        if i is odd:
            divisorLon = 64
            divisorLat = 32
        else:
            divisorLon = 32
            divisorLat = 64
        
        // Calculate row and column
        column = index mod divisorLon
        row = integer(index / divisorLon)
        
        // Update ranges
        widthLon = (maxLon - minLon) / divisorLon
        heightLat = (maxLat - minLat) / divisorLat
        
        minLon = minLon + column * widthLon
        maxLon = minLon + widthLon
        minLat = minLat + row * heightLat
        maxLat = minLat + heightLat
    
    // Calculate the center of the final plot
    longitude = (minLon + maxLon) / 2
    latitude = (minLat + maxLat) / 2
    
    return [latitude, longitude]
```

## 8. Conclusions

The BGrid coordinate system offers an innovative alternative to traditional systems, with a special emphasis on usability, verbal communication, and adaptability to different precision needs. Its integration with the BIP39 standard facilitates the memorization and transmission of geographic locations, while its hierarchical structure allows adjusting the precision according to the requirements of each application.

The alternation in division factors between levels contributes to maintaining a square shape for the plots, which partially mitigates the distortion inherent in projecting a spherical surface onto a rectangular coordinate system.

## References

1. Bitcoin Improvement Proposal 39 (BIP39): [https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki)
2. Geographic coordinate systems. National Geographic Institute.
3. Sahr, K., White, D., & Kimerling, A. J. (2003). Geodesic discrete global grid systems. Cartography and Geographic Information Science, 30(2), 121-134.
4. What3Words: An alternative geographic encoding system. [https://what3words.com/about](https://what3words.com/about)
5. Geohash: Public geocoding system. [https://en.wikipedia.org/wiki/Geohash](https://en.wikipedia.org/wiki/Geohash)

## 9. License
Apache 2.0  
http://www.apache.org/licenses/
