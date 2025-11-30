# Operation Pensieve Breach – 1 (Windows / DCsync IR) Writeup

**Flag:**

```text
Hero{albus.dumbledore;22/11/2025-23:13:41;192.168.56.200;192.168.56.230}
```

---

## 0. Challenge Recap

We’re given Windows event logs from the **Principal Domain Controller** of the Ministry of Magic. The story:

> A critical user has been compromised and is performing nasty magic using the DCsync spell.

We must extract **four** things from the DC’s logs:

1. `sAMAccountName` (lowercase) of the compromised account performing bad stuff.  
2. **Timestamp of the beginning of the attack**, format:  
   `DD/MM/YYYY-HH:MM:SS` (local SystemTime, not literally the word `SystemTime`).  
3. **Source IP address** used for the attack (i.e., the DCsync source IP).  
4. **Last legitimate IP** used to login **by any user** before the DCsync started.  

And return them as:

```text
Hero{user;DD/MM/YYYY-HH:MM:SS;attack_ip;last_legit_ip}
```

---

## 1. Tools & Parsing

We work with the **Security.evtx** from the DC.

### Parsing the log

On Windows, we can use **EvtxECmd** (Eric Zimmerman’s tool) to convert EVTX to CSV:

```cmd
EvtxECmd.exe -f "C:\lab\logs\Security.evtx" --csv "C:\lab\parsed" --csvf Security.csv
```

This produces:

```text
C:\lab\parsed\Security.csv
```

All further analysis is based on that `Security.csv`.

We then load it (Excel, Timeline Explorer, or Python) and verify key columns:

- `EventId`
- `TimeCreated` (local machine time)
- `SubjectUserName`, `SubjectDomainName`
- `TargetUserName`, `TargetDomainName`
- `SubjectLogonId`, `TargetLogonId`
- `IpAddress`
- Other payload fields (AccessMask, Properties, etc.)

---

## 2. Detecting DCsync in Security Logs

A **DCsync** attack abuses normal AD replication to pull password hashes from a Domain Controller. On Windows Security logs, this usually appears as:

- **Event ID 4662** – “An operation was performed on an object”
- `ObjectServer = DS`
- `AccessMask = 0x100` (Control Access)
- `Properties` containing AD replication rights GUIDs, especially:
  - `1131f6aa-9c07-11d1-f79f-00c04fc2dcd2` → DS-Replication-Get-Changes  
  - `1131f6ad-9c07-11d1-f79f-00c04fc2dcd2` → DS-Replication-Get-Changes-All  
  - `89e95b76-444d-4c62-991a-0facbeda640c` → DS-Replication-Get-Changes-In-Filtered-Set  

### 2.1 – Filter 4662 events

In the CSV:

1. Filter `EventId = 4662`.
2. Within those events, filter `AccessMask = 0x100`.
3. Within those, search the `Properties` (or equivalent payload fields) for the replication GUIDs:
   - `1131f6aa`
   - `1131f6ad`
   - `89e95b76`

This gives all **DCsync-like operations**.

### 2.2 – Who’s doing the DCsync?

Among those 4662 events we see two main subjects:

- `SubjectUserName = MINISTRY$` – the DC’s own machine account (legitimate replication).
- `SubjectUserName = albus.dumbledore` – a user account (our malicious DCsync operator).

So the **compromised account actually performing DCsync is:**

```text
albus.dumbledore
```

This directly answers:

> sAMAccountName (lowercase) of the compromised account performing bad stuff → `albus.dumbledore`

---

## 3. Finding When the DCsync Started

The author clarified:

> “Timestamp of the beginning of the attack”  
> → **The timestamp where the DCsync started (local machine time).**

So we just need the **first 4662 DCsync event** for `albus.dumbledore`.

### 3.1 – Filter DCsync events for albus

From the DCsync 4662 set, filter:

- `SubjectUserName = albus.dumbledore`

Then sort those rows by `TimeCreated` ascending.

The earliest event looks like:

```text
EventId        : 4662
SubjectUserName: albus.dumbledore
AccessMask     : 0x100
Properties     : ... {1131f6aa...}, {1131f6ad...}, ...
TimeCreated    : 2025-11-22 23:13:41.3159347
SubjectLogonId : 0x420B75
...
```

So the **first DCsync** by the compromised user `albus.dumbledore` starts at:

```text
2025-11-22 23:13:41.3159347 (local)
```

The challenge wants `DD/MM/YYYY-HH:MM:SS` (seconds precision, no word `SystemTime`), so we format it as:

```text
22/11/2025-23:13:41
```

This is our **“beginning of the attack”** value.

---

## 4. Source IP Used for the DCsync

The 4662 event itself usually doesn’t show the source IP, but it has a **LogonId**. We can correlate that with 4624 (logon) events.

From that first DCsync 4662, we note:

```text
SubjectLogonId = 0x420B75
```

### 4.1 – Correlate with 4624

In `Security.csv`:

- Filter `EventId = 4624`.
- Search for `TargetLogonId = 0x420B75`.

We find a corresponding event:

```text
EventId        : 4624
TimeCreated    : 2025-11-22 23:13:41.2170939
TargetUserName : albus.dumbledore
TargetDomainName: hogwarts
LogonType      : 3
IpAddress      : 192.168.56.200
...
```

Thus, the **DCsync session (LogonId 0x420B75)** originates from:

```text
192.168.56.200
```

This IP is used by the attacker box to perform DCsync.

> **Source IP address used for this attack:** `192.168.56.200`

---

## 5. Last Legitimate IP Before DCsync

This part relied on the admin’s clarification:

> “The last legitimate IP used to login before the attack”  
> → **Last legitimate IP used by any user before the DCsync started.**

So:

- We must look at **all** logon events (4624), not only the compromised user.
- Consider **only events before** the first DCsync timestamp:  
  `TimeCreated < 2025-11-22 23:13:41.3159347`.
- Among those, we need the **last legitimate IP**.

We treat:

- The attacker IP `192.168.56.200` as *not* legitimate.
- The DC’s own IPs / loopbacks (`::1`, `10.0.2.15`, `fe80::...`) as internal, not client “legit” IPs.

### 5.1 – Collect 4624 before DCsync

From `Security.csv`:

1. Filter `EventId = 4624`.
2. Keep only rows where `TimeCreated < 2025-11-22 23:13:41.3159347`.
3. Sort by `TimeCreated` ascending.

Now inspect the **last few** events just before DCsync.

The final “client-style” logon from a non-attacker IP before the DCsync burst is:

```text
TimeCreated    : 2025-11-22 23:11:29.7866694
TargetUserName : pensive.svc
IpAddress      : 192.168.56.230
```

Between 23:11:29 and 23:13:41:

- We see MINISTRY$ logons from `::1`, `10.0.2.15`, `fe80::...` (internal DC activity).
- Then ANONYMOUS LOGON and `albus.dumbledore` from `192.168.56.200` (attacker).
- Finally, the DCsync events themselves.

So the **last legitimate IP used by any user before DCsync** is:

```text
192.168.56.230
```

> **Last legitimate IP used to login before the attack:** `192.168.56.230`

---

## 6. Final Flag Construction

We now have all four required values:

1. Compromised account performing DCsync (sAMAccountName, lowercase):  
   → `albus.dumbledore`
2. Timestamp where DCsync started (beginning of the attack, local time):  
   → `22/11/2025-23:13:41`
3. Source IP used for the DCsync attack:  
   → `192.168.56.200`
4. Last legitimate IP used to login by any user before DCsync:  
   → `192.168.56.230`

So the final flag is:

```text
Hero{albus.dumbledore;22/11/2025-23:13:41;192.168.56.200;192.168.56.230}
```

---

## 7. Kill Chain – High-Level Story

Even though the flag only needs four values, the lab was built to look like a real engagement:

1. **Normal domain usage**  
   - Service accounts like `pensive.svc` log in from legitimate servers (`192.168.56.230`).
   - The DC account `MINISTRY$` has regular replication and internal logons from `::1`, `10.0.2.15`, `fe80::...`.

2. **Attack foothold and staging**  
   - The attacker compromises a privileged user (`albus.dumbledore`) and starts logging in from **192.168.56.200**.
   - We see ANONYMOUS LOGON and `albus.dumbledore` sessions from that IP.

3. **Privilege abuse / DCsync**  
   - From logon session `0x420B75` (on 192.168.56.200), the attacker performs DCsync:
     - Event ID 4662, `AccessMask = 0x100`, replication GUIDs appear at `2025-11-22 23:13:41.315...`.
   - That’s when the DC starts handing over directory secrets to the attacker host.

4. **Why the specific IPs matter**  
   - `192.168.56.200` → the attack workstation performing DCsync.  
   - `192.168.56.230` → the last “legitimate” machine to log in before the DCsync began.  
