# Float30

30-key hand-held column-staggered split wireless keyboard. ZMK firmware on Nice!Nano v2.

## KiCAD MCP Server

This project uses the [KiCAD MCP Server](https://github.com/anomalyco/KiCAD-MCP-Server) to interact with KiCAD via AI assistants. It is configured in [`opencode.json`](./opencode.json).

### Setup Summary

The MCP server spawns a Python subprocess that uses KiCAD's `pcbnew` module. Here is everything needed for it to work:

| What | Value |
|---|---|
| **Python** | 3.14.2 (via `uv`, pinned in `.python-version`) |
| **Venv** | `.venv/` (created with `uv venv --python 3.14 --system-site-packages`) |
| **pcbnew** | System KiCAD 9.0.9 at `/usr/lib64/python3.14/site-packages/pcbnew.py` |
| **MCP server** | `KiCAD-MCP-Server/dist/index.js` (Node.js) |
| **Symlink** | `KiCAD-MCP-Server/.venv` → `.venv` (so the server auto-discovers the venv Python) |
| **.pth file** | `.venv/lib/python3.14/site-packages/kicad.pth` contains `/usr/lib64/python3.14/site-packages` |

### How the MCP server finds Python

The `findPythonExecutable` function in the MCP server checks these locations in order:

1. `KiCAD-MCP-Server/.venv/bin/python` ← **the symlink we created satisfies this**
2. `KICAD_PYTHON` environment variable (also set in `opencode.json`)
3. System fallbacks (`which python3`, etc.)

The `.venv` symlink at the MCP server root is the primary mechanism. If it ever breaks, the `KICAD_PYTHON=~/.venv/bin/python` env var in `opencode.json` is the backup.

### Verifying the MCP server works

From the project root, run these quick checks:

```bash
# Check pcbnew is importable from the venv Python
.venv/bin/python -c "import pcbnew; print(pcbnew.Version())"

# Test the Python backend directly (send a JSON command via stdin)
printf '{"command":"get_backend_state","id":"1"}\n' \
  | .venv/bin/python KiCAD-MCP-Server/python/kicad_interface.py 2>/dev/null

# Test opening the board
printf '{"command":"open_project","params":{"filename":"'$(pwd)'/pcb/charybdis.kicad_pcb"},"id":"2"}\n' \
  | timeout 10 .venv/bin/python KiCAD-MCP-Server/python/kicad_interface.py 2>/dev/null
```

If the MCP tools are unavailable in your session, check `opencode mcp list` — the kicad server should show as "connected". Use `opencode.json` toggling (`enabled: false` → `enabled: true`) and an opencode restart if needed.

### Recreating the venv from scratch

If the `.venv` or anything breaks:

```bash
rm -rf .venv
uv python install 3.14
uv venv --python 3.14 --system-site-packages
uv sync
echo "/usr/lib64/python3.14/site-packages" > .venv/lib/python3.14/site-packages/kicad.pth
ln -sf "$(pwd)/.venv" KiCAD-MCP-Server/.venv
```

### Why Python 3.14 (not 3.12)

The system KiCAD install provides `_pcbnew.so` compiled for Python 3.14 (`libpython3.14.so.1.0`). Using Python 3.12 would cause ABI mismatches during board operations (imports appear to work but actual calls crash or hang). The venv was downgraded from >=3.12 to >=3.14 specifically to match the system KiCAD Python bindings.

## Board Files

| File | Description |
|---|---|
| `pcb/charybdis.kicad_pcb` | Main PCB layout |
| `pcb/flex.kicad_pro` | Secondary project file |
| `pcb/flex.kicad_prl` | Project-local library |
| `pcb/flex-backups/` | Backups |

KiCAD 9.0.9 is installed on this system (`/usr/bin/kicad-cli`).

## Dependencies

Managed with `uv`. See [`pyproject.toml`](./pyproject.toml):

- `kicad-python>=0.7.1` — KiCAD API bindings
- `kicad-skip>=0.2.5` — S-expression KiCAD file parser
- `cairosvg`, `pillow` — rendering
- `sexpdata`, `pynng`, `python-dotenv`, `requests`
