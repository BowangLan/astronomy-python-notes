# SPICE

- PPT Slides from NASA: [An Overview of SPICE](https://naif.jpl.nasa.gov/pub/naif/toolkit_docs/Tutorials/pdf/individual_docs/03_spice_overview.pdf)

- A list of all SPICE function docs: [CSPICE functions](https://naif.jpl.nasa.gov/pub/naif/toolkit_docs/C/cspice/index.html)

## Install

```bash
pip install spiceypy
```

## Kernel

Most SPICE functions require a _kernel_ file to complete calculate.

NASA and ESA provide repositories containing a lot of Kernels. For example:

- https://naif.jpl.nasa.gov/pub/naif/

There are different kinds of Kernel files, distinguished by their file extension:

- `.spk` contain trajectory information of planetary bodies, spacecraft, etc.
- `.pck` contain physical parameters of bodies like the size or orientation
- `.ik` contain instrument-specific information that are e.g., mounted on a spacecraft
- `.ck` contain information regarding the orientation of a spacecraft in space
- `.fk` contain reference frame information that is needed to calculate positions in a less common reference system
- `.lsk` contain time information that is crucial to convert e.g., the UTC time into ephemeris time ET (a standard time format that is being used in space science and astronomy)

How to load the a kernel file:

```python
spiceypy.furnsh(filepath)
```

## Calculate the Earth's Position and Velocity

Convert a datetime string from `%Y-%m-%dT%H:%M:%S` to ET:

- Function: `spiceypy.utc2et(utf_datetime_str)`
- Kernel required: [naif0012.tls](https://naif.jpl.nasa.gov/pub/naif/generic_kernels/lsk/naif0012.tls)

Example:

```python
import datetime

# convert the datetime to a string, replacing the time with midnight
today_dt_str = datetime.datetime.today().strftime('%Y-%m-%dT00:00:00')

# convert the utc midnight string to the corresponding ET
today_et = spiceypy.utc2et(today_dt_str)
```

---

Convert km to AU:

- `spiceypy.convrt()`

```python
spiceypy.convrt(dst_km, 'km', 'AU')

```

- [NAIF Integer ID code](https://naif.jpl.nasa.gov/pub/naif/toolkit_docs/C/req/naif_ids.html)
  - includes a complete list of NAIF ID codes

---

NAIF ID codes for common objects:

- 10 - Sun
- 399 - Earth

---

Kernel required for computing the position and velosity of Earth w.r.t. the Sun today:

- `https://naif.jpl.nasa.gov/pub/naif/generic_kernels/spk/planets/de432s.bsp`

A summary of the files can be found in `aa_summaries.txt`.

---

`spiceypy.spkgeo`

- [spkgeo docs](https://naif.jpl.nasa.gov/pub/naif/toolkit_docs/C/cspice/spkgeo_c.html)

> Compute the geometric state (position and velocity) of a target body relative to an observing body.

Parameters:

- `targ` (int) - the standard NAIF ID code for a target body.
- `et` (float) - the epoch at which the state of the target body is to be computed.
- `ref` (str) - the name of the reference frame
- `obs` (int) - the standard NAIF ID code for the observing body.

Output: `(state, lt)`

- `state` - contains the geometric position and velocity of the target body, relative to the observing body, at epoch `et` .
  ```python
  pos_vct = state[:3]
  vel_vct = state[3:]
  ```
- `lt` - the one-way light time from the observing body to the geometric position of the target body in seconds at the specified epoch.

Example:

```python
targ: int = 399 # the Earth
et: float = today_et
ref: str = 'ECLIPJ2000'
obs: int = 10

state, lt = spiceypy.spkgeo(targ, et, ref, obs)

distance = np.sqrt(np.sum(state[:3]**2))
dist_au = spiceypy.convrt(distance, 'km', 'AU')
print("The distance between the Sun and the Earth right now is {} km or {} AU".format(distance, dist_au))

velocity = np.sqrt(np.sum(state[3:]**2))
print("The velocity of the Earth relative to the Sun is {} km/s".format(velocity))
```

Another way to calculate the velocity of the Earth.