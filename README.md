# reachy-mini-sim

A pinned environment that runs the
[Reachy Mini](https://huggingface.co/docs/reachy_mini) daemon against a
[MuJoCo](https://mujoco.org) simulation, so
[`reachy-mini-mcp`](../reachy-mini-mcp) can be developed without the robot
plugged in.

The simulated daemon serves the same REST API on the same port as a real
Reachy Mini Lite. The MCP server therefore needs no change and no environment
variable: its default `http://localhost:8000` already points here.

## What this project is

No code of its own — `pyproject.toml` and `uv.lock` pin one reproducible
daemon environment. `[tool.uv] package = false` says so explicitly.

## Requirements

- Python 3.12
- [uv](https://docs.astral.sh/uv/)

Windows, macOS and Linux are supported. On Linux, GStreamer must be installed
manually; see the
[SDK installation guide](https://huggingface.co/docs/reachy_mini/SDK/installation).

## Installation

```bash
uv sync
```

## Running the simulation

```bash
uv run reachy-mini-daemon --sim
```

A window opens with the 3D view of the robot: drag to rotate, right-click to
pan, scroll to zoom. The daemon serves its REST API on `http://localhost:8000`
and binds to `127.0.0.1` only.

On macOS, MuJoCo needs its own launcher:

```bash
uv run mjpython -m reachy_mini.daemon.app.main --sim
```

### Useful flags

| Flag              | Effect                                                          |
| ----------------- | --------------------------------------------------------------- |
| `--scene minimal` | Adds a table and objects (apple, croissant, duck) to the scene   |
| `--headless`      | Runs MuJoCo without the 3D window                                |
| `--no-media`      | Disables camera, audio and WebRTC; the daemon leaves the mic alone |
| `--mockup-sim`    | Skips MuJoCo entirely: targets apply instantly, no physics, no window |

`--mockup-sim` replaces `--sim` rather than accompanying it. It serves the
same REST API at a fraction of the CPU cost, which matters on battery — what
it drops is the physics and the visual check.

## Working on the MCP server against the simulation

Start the daemon here, then run the MCP server from its own directory as
usual. Its `get_daemon_status` tool reports `reachable: true` and
`is_ready: true` once the simulated backend is running.

To confirm the daemon is up without an MCP client:

```bash
curl http://localhost:8000/api/daemon/status
```

The daemon also serves interactive API documentation at
<http://localhost:8000/docs>, which is the quickest way to find the endpoints
for the next MCP tool.

## Version pinning

`reachy-mini` is constrained to `>=1.9,<2` and locked to an exact version in
`uv.lock`. Keep this in step with the daemon version `reachy-mini-mcp` targets
in its tests, so a passing test suite means the same thing here and on the
real robot.
