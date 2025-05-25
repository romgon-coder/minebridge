# minebridge
A cross-version save-bridging tool to keep a single Minecraft world playable across multiple game versions without data corruption.
1. What MineBridge Does

* **Automated Version-Specific Copies**
  Creates and maintains separate copies of a given world for each target Minecraft version in a `bridged_saves/` folder.

* **Seamless Switching**
  Uses symbolic links so that the Minecraft launcher always sees the same world name, while pointing to the correct version-specific files.

* **Safe Backups**
  Before any modification, automatically backs up the original world to `saves_backup/` to prevent accidental data loss.

* **DataVersion Conversion**
  Parses each version’s `version.json` for its `DataVersion` tag and patches `level.dat` via `nbtlib` to ensure compatibility with newer and older clients.

* **Dual Interfaces**

  * **CLI**: Full command-line interface for automations and scripting.
  * **GUI**: Lightweight Tkinter front-end for point-and-click world bridging.

* **Packaging-Ready**

  * Installable via PyPI (with `setup.py`)
  * Standalone executables via PyInstaller

---

## 2. How MineBridge Works

### 2.1 Directory Structure

```
.minecraft/
├── versions/                # Installed game versions (1.16.5, 1.18.2, …)
├── saves/                   # Default world folder
│   └── MyWorld/             # Original world (symlink to bridged_saves/MyWorld_1.18.2)
├── bridged_saves/           # Versioned world copies
│   ├── MyWorld_1.16.5/
│   └── MyWorld_1.18.2/
└── saves_backup/            # Backups of original worlds (timestamped or versioned)
```

### 2.2 Bridging Process (per version)

1. **Copy**: Duplicate `<saves>/<world>` into `bridged_saves/<world>_<version>`.
2. **Backup**: Move original `<saves>/<world>` into `saves_backup/` (if not already backed up).
3. **Symlink**: Create a symlink at `<saves>/<world>` pointing to the versioned folder.
4. **Convert**: Load and rewrite `level.dat` with the correct `DataVersion` tag.

### 2.3 DataVersion Conversion

* Reads `<minecraft-dir>/versions/<version>/<version>.json`.
* Extracts `DataVersion` field.
* Uses `nbtlib` to load `level.dat`, update its tag, and save back.

---

## 3. User Interfaces

### 3.1 Command-Line Interface (CLI)

```
# List available worlds
mc-bridge --list-worlds

# List installed game versions
mc-bridge --show-versions

# Bridge 'MyWorld' for version 1.18.2 and launch Minecraft
mc-bridge --world MyWorld --version 1.18.2 --launch
```

### 3.2 Graphical UI (Tkinter)

* Launch with no arguments: `mc-bridge` or `python minecraft_version_bridge.py`
* Dropdowns to select world and version, checkbox to launch after bridging.

---

## 4. Packaging & Distribution

* **PyPI Package**: via `setup.py` and `requirements.txt`
* **Standalone Binaries**: via PyInstaller for Windows (.exe), macOS (.app), Linux (AppImage/.deb/.rpm)
* **Auto-Build Workflow**: GitHub Actions to produce release artifacts on tagging.

---

## 5. Dependencies

* **nbtlib**: For reading/writing Minecraft NBT data (`pip install nbtlib`).
* **tkinter**: Standard Python GUI library (bundled with most Python installs).
* **PyInstaller** (optional): For binary packaging.

---

## 6. Similar Tools & Comparison

| Tool                    | Purpose                               | Notes                                             |
| ----------------------- | ------------------------------------- | ------------------------------------------------- |
| **MCEdit / Amulet**     | World editing and chunk manipulation  | Manual editing; not automated bridging.           |
| **Universal MC Editor** | GUI-based NBT and world editing       | Windows-only; manual save modifications.          |
| **RegionFixer**         | Repairs corrupted region files        | Focuses on chunk errors, not versioning.          |
| **MultiMC**             | Multi-instance launcher & mod manager | Manages instances, but copies saves per instance. |

**MineBridge Unique Value:**

* Fully automated, reversible bridging across versions.
* Automatic DataVersion patches for backwards/forwards compatibility.
* Lightweight—no heavy GUI editor, just symlinks and NBT tweaks.
