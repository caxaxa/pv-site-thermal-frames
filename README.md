# PV Site Thermal Frames with Radiometric Pixel Temperatures (public subset, v1.0)

Raw 16-bit thermal frames from drone flights over photovoltaic plants, with the temperature the camera
manufacturer's SDK computes at sampled pixels of each frame. Licensed under CC BY 4.0.

## Overview

Nadir thermal frames from drone inspection flights over photovoltaic plants, captured with a DJI M30T thermal camera
at about 25-50 m above the take-off point, between 10:02 and 13:28 on the camera clock. Each frame is the camera's
raw 16-bit sensor count image as stored inside the radiometric JPEG.

For every frame the dataset gives six pixel positions and the temperature that the manufacturer's SDK computes for
each of them. The positions are chosen by a seeded program, not by people: three are local maxima of the lightly
smoothed temperature image inside the hottest 1 percent of the frame (kind `hot`), and three are uniformly random
pixels (kind `random`), all at least 12 pixels apart (Euclidean distance) and 8 pixels from the border.

This release contains 315 frames from 4 flights over 3 site-days and 1,890 pixels (945 hot, 945 random).

## Intended use

Work on radiometric calibration and thermal measurement from raw sensor data, for example:

- learning the mapping from raw sensor counts to temperature for a camera model;
- estimating the per-frame offset that the manufacturer's processing removes using calibration data stored in the
  original camera file (see Known limitations);
- studying how that offset behaves across flights, site-days and time within a flight;
- benchmarking measurement accuracy under shift between site-days.

It is not intended for hotspot detection or localisation (the pixels are not a census of the hot cells in a frame
and no person confirmed them), for PV fault diagnosis or severity grading (no electrical ground truth, no
module-level labels), or for absolute-accuracy claims about the camera (the temperatures are computed values, not
traceable reference measurements).

## Files

| file | contents | sha256 |
|---|---|---|
| `frames/frames_F00a32e.tar` | raw frames of flight F00a32e (PNG, tar) | `b3a4e0f0831720c59654cd3917b72386b9bccfd9a346b2d31732b21871d491dd` |
| `frames/frames_F64732a.tar` | raw frames of flight F64732a (PNG, tar) | `cd05a861a95148ae385eb6cf6fda5b0eb119701c00fca66703ae832263f32c7a` |
| `frames/frames_Fe31007.tar` | raw frames of flight Fe31007 (PNG, tar) | `c4d3fd993213eaf22c13d688daee258d81843ef6c2cb52b5d7aa5ccd6ffbc1b1` |
| `frames/frames_Ff59812.tar` | raw frames of flight Ff59812 (PNG, tar) | `be1f70cbb9715db118fb38126240cb1938fa8259ca191417635b1196a6c316b6` |
| `frames.csv` | frame index (315 rows) | `f752f917f97897c7f3343a7232ae9768edf0d51baf18c3ecd15337fa18ca580a` |
| `points.csv` | pixel temperatures (1,890 rows) | `e637132c3a02c3d5fc245740e74ad194f3d773712419859d7ccb17ff3a46be03` |
| `LICENSE` | CC BY 4.0 legal code | `9ba9550ad48438d0836ddab3da480b3b69ffa0aac7b7878b5a0039e7ab429411` |

Unpack the frames with `mkdir -p frames_png && for t in frames/*.tar; do tar -xf "$t" -C frames_png; done`; each
archive holds `<frame_id>.png` for one flight. `SHA256SUMS` lists the same checksums
(`sha256sum -c SHA256SUMS`).

- Each PNG is a 640x512 single-channel 16-bit image (unsigned 16-bit integer pixel values) holding raw sensor
  counts, with no metadata chunks. `frames.csv` is the authoritative index: one PNG per row, none outside it.

### Columns of frames.csv (315 rows)

- `frame_id` (string): frame identifier, a salted hash; matches the PNG file name.
- `flight` (string): flight identifier, a salted hash; 4 distinct values, 45 to 90 frames per flight.
- `session` (string): site-day identifier, a salted hash; 3 distinct values. Frames that share a session were taken
  at the same site on the same day.
- `camera` (string): camera model, M30T for every frame.
- `capture_time` (string): time of day on the camera clock, HH:MM:SS. No date is given.
- `altitude_m` (float): height above the take-off point in metres, 24.90 to 50.03.

### Columns of points.csv (1,890 rows, six per frame)

- `sample_id` (string): identifier of one pixel of one frame, a salted hash.
- `frame_id` (string): the frame this pixel lies in.
- `kind` (string): `hot` for a local maximum in the hottest 1 percent of the frame, or `random` for a uniformly drawn
  pixel.
- `x` (integer): pixel column, 0 to 639 (8 to 631 observed).
- `y` (integer): pixel row, 0 to 511 (8 to 503 observed).
- `temp_c` (float): the manufacturer's SDK temperature of that pixel in the same frame, in degrees Celsius, on a
  0.1 degC grid. Range 6.1 to 65.2; median 40.3 (hot 46.3, random 35.2).

## Collection and processing

Frames come from drone inspection flights over photovoltaic plants and were decoded from the original radiometric
JPEG files with the manufacturer's processing (the DJI Thermal SDK). Both the raw count image and the temperature
image come from that processing, so the temperatures are consistent with the counts frame by frame.

Pixel selection is by program: per frame, the temperature image is smoothed with a 3x3 mean, the pixels that are the
maximum of their 7x7 neighbourhood and lie in the top 1 percent of the frame (border excluded) are candidates, and
three are taken in a seeded random order with a minimum spacing of 12 pixels; three random pixels are then drawn
uniformly under the same spacing and border rules, also away from the hot ones. The seed of each frame is a salted
hash. The temperature of a chosen pixel is read from the unsmoothed temperature image.

De-identification: original file names, capture dates, GPS coordinates and EXIF, device serial numbers, and plant,
client, project and user identifiers are removed, and the camera's stored measurement parameters are not shipped as
columns (their constant values are stated under Known limitations). All identifiers are salted hashes; the salt is
not published, so they cannot be recomputed from the original names. The images show module rows and the ground
around them and, in some frames, buildings, equipment or vehicles. The frames are thermal, taken looking straight
down from 25 to 50 m at 640x512 pixels; no name, coordinate or identifier of any person is released.

## Known limitations

- The raw counts are not corrected for a per-frame offset. The temperatures are the SDK's output, and its processing
  appears to handle that offset with calibration data stored in the original camera file, which is not part of this
  dataset. Its cause is not measured here, only its size and behaviour: the temperature a frame assigns to a fixed
  count varies by several degrees across frames, and within a flight it changes with time in ramps that end in
  abrupt drops at irregular intervals.
- Absolute radiometric accuracy is limited. Manufacturer specifications for cameras of this class are typically the
  greater of about 2 degC or 2 percent of the reading. The temperatures are computed values, never validated against
  contact thermometry.
- The camera's measurement parameters were identical in every frame (5 m object distance, 60 percent humidity,
  emissivity 0.95, reflected temperature 23 degC, ambient air temperature setting 0); they are not shipped as
  columns. Real module emissivity varies with glass, soiling and view angle, and the true distance was about 25-50 m,
  so absolute temperatures carry a systematic bias that varies with surface and geometry.
- The pixels are not a census. Hot pixels are local maxima of the frame wherever they fall, on a module or on any
  other surface; no person checked what they sit on. Random pixels follow the area of each surface type in the frame.
- Small number of site-days and flights (3 site-days, 4 flights, one camera model), so per-session statistics are
  unstable.

## Licence and provenance

- Licence: CC BY 4.0 (see `LICENSE`) for the frames, the two tables and this description. Credit the dataset as
  "PV Site Thermal Frames with Radiometric Pixel Temperatures" and the licensor below. The licence does not extend to
  the manufacturer's SDK, which is not shipped.
- Licensor and publisher: LUCAS A. CHACHA CONSULTORIA E ANÁLISE DE DADOS LTDA, a company registered in Brazil
  (CNPJ 48.786.365/0001-10).
- Origin: original data, flown and captured by the publisher; not a re-upload of another dataset.
- Right to release: the publisher holds the right to release this imagery and does so in its own name as licensor.
  To the publisher's knowledge no client agreement or other contract prevents this release, and no third party holds
  rights in the frames.
- Labels: no person marked or labelled anything. Pixel positions come from a seeded program and temperatures are the
  camera manufacturer's SDK output for the frame, computed by the publisher under its licence for the SDK from its
  own images.
- Withdrawal: anyone who believes they hold rights in a flight shown here can contact the publisher through its
  public registry entry (CNPJ above). The publisher will stop distributing that flight. Copies already received under
  CC BY 4.0 cannot be recalled; that licence is irrevocable.

## Citation

LUCAS A. CHACHA CONSULTORIA E ANÁLISE DE DADOS LTDA (2026). PV Site Thermal Frames with Radiometric Pixel
Temperatures (public subset, v1.0). CC BY 4.0. https://github.com/caxaxa/pv-site-thermal-frames
