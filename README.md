
## 1. What MineBridge Does

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

### 4.1 GitHub Repository Setup

To centralize development and allow collaboration:

1. **Initialize your local Git repo**

   ```bash
   git init
   git branch -m main
   git add .
   git commit -m "Initial commit: scaffold MineBridge project"
   ```

2. **Create the remote repository**

   * **Via GitHub CLI**:

     ```bash
     gh repo create NikolaosAngelosoulis/minebridge \
       --public --source=. --remote=origin --push
     ```
   * **Via GitHub Web UI**:

     1. Go to [https://github.com/new](https://github.com/new)
     2. Set repository name to `minebridge`, make it Public
     3. Skip README upload (we have one), then click **Create repository**
     4. Follow the displayed push commands to link and push your local repo.

3. **Configure repository settings**

   * Under **Settings > Manage Access**, invite collaborators by GitHub username or email.
   * Enable **Issues**, **Pull Requests**, and **Actions** for CI/CD.
   * Add branch protection rules for `main` (e.g., require PR reviews).

4. **Ongoing maintenance**

   * Use labels and issue templates for bug reports and feature requests.
   * Set up a **CONTRIBUTING.md** to guide new contributors on coding standards, testing, and commit message format.

### 4.2 Packaging & Distribution

* **PyPI Package**: via `setup.py` and `requirements.txt`
* **Standalone Binaries**: via PyInstaller for Windows (.exe), macOS (.app), Linux (AppImage/.deb/.rpm)
* **Auto-Build Workflow**: GitHub Actions to produce release artifacts on tagging.

#### 4.3 Distributing a ZIP Bundle

You can package a full runnable build—including executables, assets, and folder structure—into a single ZIP for users:

1. **Build Binaries**

   * Run PyInstaller to produce platform-specific executables:

     ```bash
     pyinstaller --onefile --name minebridge minebridge.spec  # Uses preconfigured spec
     ```
   * Artifacts appear under `dist/`:

     ```text
     dist/
     ├─ minebridge.exe       # Windows
     ├─ minebridge           # Linux
     └─ minebridge.app       # macOS
     ```

2. **Organize Folders**
   Create a staging directory with this layout:

   ```text
   minebridge-<version>-<platform>/
   ├── bin/                 # Executable(s)
   │   ├── minebridge.exe   # or 'minebridge' on Unix
   ├── README.md
   ├── LICENSE
   └── config/             # Optional default config (if any)
   ```

3. **Zip It Up**
   From the parent of `minebridge-<version>-<platform>/`, run:

   ```bash
   zip -r minebridge-<version>-<platform>.zip minebridge-<version>-<platform>/
   ```

4. **Running from ZIP**
   Users can unzip and execute:

   ```bash
   unzip minebridge-2.0.0-windows.zip
   cd minebridge-2.0.0-windows/bin
   ./minebridge.exe --help
   ```

   On Unix, mark the binary as executable if needed:

   ```bash
   chmod +x minebridge
   ./minebridge --world MyWorld --version 1.18.2
   ```

This ZIP distribution is self-contained: no Python install or dependencies are required on the target machine.

## 5. Similar Tools & Comparison

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
