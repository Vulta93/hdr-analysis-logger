# HDR Analysis Logger

A ReShade add-on that writes the measurements of **Lilium's HDR/SDR analysis**
effect to JSON files, so you get numbers instead of screenshots of the overlay.
It can also measure a box you draw around a single light source.

Built for HDR modding: A/B tests (vanilla vs. mod, slider changes), checking
peaks and averages over time, or handing measurements to scripts and AI tools.

> **Credit:** all whole-frame statistics are computed by
> [Lilium's ReShade HDR shaders](https://github.com/EndlesslyFlowering/ReShade_HDR_shaders)
> (`lilium__hdr_and_sdr_analysis.fx`). This add-on does not modify or include
> that shader; it only reads back the small texture where the effect stores its
> results. If the effect's layout changes in a future version, the add-on
> reports it instead of logging wrong values.

## Features

- **Whole-frame stats from Lilium's analysis:** max/avg/min luminance, max/avg/min
  R, G, B and CLL, and gamut percentages (BT.709 / DCI-P3 / BT.2020, plus AP0 and
  invalid in scRGB).
- **Measurement region:** a box at the screen centre, or one you draw or drag onto
  a light source. Logged separately with luminance, R/G/B and CLL. Measured
  before any ReShade effect runs, so overlays never end up in the numbers. Works
  even without Lilium's effect.
- **Series recording:** press a hotkey (default **Shift+F9**, rebindable) to log
  5 seconds at 100 ms intervals and save min/mean/max for every value. An
  on-screen banner counts down and shows when the file is written.
- Everything works with the ReShade menu closed.

## Requirements

- Windows, with ReShade **6.7.3 or newer with full add-on support**. The add-on
  is built against add-on API 18; older ReShade versions may refuse to load it.
- For the whole-frame stats: Lilium's `lilium__hdr_and_sdr_analysis.fx`, which
  supports DirectX 11, DirectX 12, OpenGL and Vulkan.

## Install

1. Download `HDRAnalysisLogger.addon64` (64-bit games) or
   `HDRAnalysisLogger.addon32` (32-bit games) from
   [Releases](../../releases).
2. Put it next to the game's ReShade DLL.
3. In ReShade, enable **Lilium's HDR analysis** and drag it to the **bottom** of
   the technique list. Techniques run top to bottom, so effects above it are
   included in its numbers and effects below it are not. For mod work it's best
   as the only active effect.

## Usage

Open the ReShade overlay and go to **Add-ons → HDR Analysis Logger**.

- **Record a series:** set a label (e.g. `vanilla-street-night`), then press the
  hotkey or **Record series**.
- **Measure one light source:** click **Draw region**. You can close the ReShade
  menu now; game input is paused while you draw. Then either drag the cyan box
  onto the target, or click one corner of a new box and then the opposite corner.
  Esc cancels. The box is fixed on screen, so keep the camera still while
  recording.
- **Change hotkey**, sample interval, series duration, the session log and the
  outline are all in the same panel and saved to `ReShade.ini`.

## Output

Files go to `<ReShade folder>\hdr_analysis\` (normally the game's folder):

| File | Contents |
|---|---|
| `series-<date-time>-<label>.json` | One recording: `summary` (min/mean/max/count per value) and all `samples`. |
| `latest.json` | The most recent sample, overwritten every sample (500 ms by default). |
| `session-<date-time>.jsonl` | Every sample of the game session, one per line (can be turned off). |

Each sample contains:

- `time`, `frame`, `game_folder`, `api`, `back_buffer_format`
- `colour_space` (`scrgb`, `hdr10`, `sdr`) and `units` (`nits` in HDR, `percent`
  in SDR)
- `values`: the whole-frame stats from Lilium. Averages are clamped to
  [min, max] like the overlay does. R/G/B are BT.2020 nits in HDR and BT.709
  percent in SDR; the max of each channel may come from a different pixel. If the
  effect is missing or disabled, `analysis_status` explains why instead.
- `active_area`: Lilium's own crop settings. While they are enabled, Lilium paints
  the cropped margins black, so only the max values stay meaningful.
- `region`: `anchor` (`screen_centre` or `drawn`), position and size in pixels,
  and its own `values`. The image is interpreted like Lilium's defaults: 10-bit as
  HDR10 PQ, 16-bit float as scRGB, 8-bit as gamma 2.2.

In a series summary, plain names are the whole frame and `region_` names are the
box.

## Limitations

- Each sample waits for the GPU to finish a tiny copy. Sampling is rate-limited,
  but it's a measurement tool, not something to leave running while playing.
- Showing the region outline keeps ReShade's GUI active every frame, which costs
  a little performance. It can be turned off.
- Not logged: Lilium's cursor readout, waveform and CIE diagram. Percentiles
  aren't available because the effect doesn't compute them.
- Tested so far in one DirectX 11 HDR10 game. scRGB, SDR and other APIs are
  implemented but not yet confirmed.

## Building

Requires CMake 3.20+ and Visual Studio 2022 (or the Build Tools) with the C++
workload. All dependencies are vendored in `third_party/`.

```
cmake -S . -B build-x64 -A x64
cmake --build build-x64 --config Release

cmake -S . -B build-x86 -A Win32
cmake --build build-x86 --config Release
```

The add-ons are written to `build-x64\Release\HDRAnalysisLogger.addon64` and
`build-x86\Release\HDRAnalysisLogger.addon32`. Each architecture needs its own
build folder. GitHub Actions builds both
architectures on every push and attaches them to a release when a `v*` tag is
pushed.

## License

MIT, see [LICENSE](LICENSE). Third-party licenses are listed in
[third_party/README.md](third_party/README.md).
