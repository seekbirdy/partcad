# PartCAD Troubleshooting Guide

Common issues and solutions for developing and using PartCAD.

## VTK Missing in Sandbox Environments

### Symptom
When inspecting PartCAD assemblies, you see errors like:
```
ERROR: fastener/raisedcheesehead-iso7045: No module named 'vtkmodules.vtkCommonDataModel'
```

The error appears in a sandbox environment path:
```
C:\Users\User\.partcad\sandbox\pc-py-conda-3.11\v-env-*\Scripts\python.exe
```

### Root Cause
PartCAD's isolated sandbox environments (used for executing CAD scripts) don't have VTK properly installed, even if it exists in the main project environment.

### Solution

⚠️ **WARNING: This is a lengthy operation (5–15 minutes)**

1. **Close VS Code completely**
   - The PartCAD extension daemon can interfere with reinstallation

2. **Reinstall PartCAD from source** (in development mode):
   ```powershell
   cd C:\_Seekbirdy_\PartCAD\partcad
   $PythonExe = "C:\_Seekbirdy_\PartCAD\PartCAD-4tv-solderless-microlab\.conda\python.exe"
   & $PythonExe -m pip install -e . --force-reinstall --no-cache-dir
   ```

3. **Clear PartCAD sandbox cache:**
   ```powershell
   Remove-Item "C:\Users\User\.partcad\sandbox\pc-py-conda-3.11" -Recurse -Force -ErrorAction SilentlyContinue
   ```

4. **Reopen VS Code**
   - The extension will restart with fresh sandbox environments

5. **Verify the fix**
   - The next time you inspect assemblies, PartCAD will rebuild sandboxes with all dependencies correctly installed

### Why This Works
- Reinstalling PartCAD from source ensures all dependencies (including VTK) are properly declared
- Clearing the sandbox cache forces PartCAD to recreate isolated environments with correct dependencies
- Fresh VS Code session avoids daemon state conflicts

### Why It's Lengthy
- `pip install -e .` with `--force-reinstall --no-cache-dir` rebuilds everything from source
- VS Code extension needs to fully restart and reinitialize the PartCAD daemon

### Faster Alternative
If you encounter this issue again, try clearing just the sandbox cache first (faster but may not always work):
```powershell
Remove-Item "C:\Users\User\.partcad\sandbox\pc-py-conda-3.11" -Recurse -Force -ErrorAction SilentlyContinue
```
Only do the full reinstall if cache clearing doesn't resolve the issue.

---

## Environment Notes

### PartCAD Sandbox vs. Project venv
**PartCAD Sandbox** (`C:\Users\User\.partcad\sandbox\...`)
- Isolated runtime environment for executing CAD scripts (CadQuery, build123d, etc.)
- Created automatically by PartCAD daemon on-demand
- Ephemeral: safe to clear/recreate without breaking anything

**Project venv** (e.g., `C:\_Seekbirdy_\PartCAD\PartCAD-4tv-solderless-microlab\.conda`)
- Main working environment for the PartCAD project
- Contains PartCAD CLI, daemon service, and project management tools
- Persistent: do not clear; contains your working installation

---

## Additional Resources
- PartCAD Documentation: `docs/source/contributing.rst`
- Project Instructions: `CLAUDE.md`
- Development Setup: `.devcontainer/devcontainer.json`
