# Temptation. Stone. Silence. — OSINT Writeup

**Category:** OSINT  
**Goal:** Identify the Latvian place name (**pilsēta/ciems**) for each of the three images and submit:

`0xfun{City1_City2_City3}`

Latvian spelling **and diacritics** are required.

---

## Overview / Approach

Each image is a “fragment” that points to a location. I treated them as three separate OSINT mini-tasks:

1. **Temptation** → find a distinctive object via reverse image search + confirm exact location.
2. **Stone** → identify the carved face/person first, then locate the matching monument/grave.
3. **Silence** → use the legend clue (deal with the devil) + the “boat” shape to locate the site.

Main techniques:
- Reverse image search (Google Images / Lens)
- Visual clue extraction (subject, environment, timestamps)
- Targeted keyword searching (person name / legend name + Latvia)

---

## 1) Temptation (Golden pig) → **Mālpils**

### What I saw
A large **golden pig statue** on grass, with flowers and a small landscaped structure nearby.

### Steps
1. **Reverse image search** the pig photo.
2. Results pointed to **two candidate locations** being discussed/appearing in results:
   - **Rīga**
   - **Mālpils**
3. The reverse search alone didn’t fully disambiguate, so I looked for **the exact photo** and any metadata.
4. The image had a **date stamp**, and with that I found a matching photo set in the **LETA** photo archive, where the **location is explicitly listed as Mālpils**.

**Reference (LETA album):**
```text
https://leta.lv/photo/album/A9CD3C76-C38D-44E6-928B-0D160A6E3101
```

✅ **City1 = Mālpils**

---

## 2) Stone (Boulder with carved face) → **Ērgļi**

### What I saw
A large boulder in a forest with a carved **portrait medallion** (face in relief) embedded into the stone.

### Steps
1. **Reverse image search** was not immediately helpful.
2. I shifted to **identifying the face**:
   - Searching for “Latvian writer portrait relief stone” and similar terms, plus visually comparing known portraits.
3. The face matches **Rūdolfs Blaumanis** (Latvian writer / playwright).
4. The challenge text says:  
   “A name important enough to be carved and left behind when everything else moves on.”  
   That strongly suggests a **grave marker / memorial stone**.
5. Searching for **Rūdolfs Blaumanis** + **grave** / **memorial** leads to a grave site in **Ērgļi**, where photos show the **same distinctive stone** with the portrait medallion.

✅ **City2 = Ērgļi**

---

## 3) Silence (Stone “boat” in forest) → **Lube**

### What I saw
A quiet forest clearing with a low, elongated stone formation resembling a **boat**.

### Steps
1. **Reverse image search** and generic AI/keyword attempts didn’t return a clean match.
2. I used the narrative clue:
   - “The silence of a legend.”
   - “They say the region’s elder once made a deal with the devil…”
3. Combining **Latvian devil folklore** + **boat**, I searched for legends/monuments with that motif.
4. This led to a known site/monument referred to as a “devil’s boat” in a forest next to **Lube**.
5. Comparing images of that site with the challenge image confirmed the match.

✅ **City3 = Lube**

---

## Final Flag

Putting the three Latvian place names together (with diacritics):

`0xfun{Mālpils_Ērgļi_Lube}`

---