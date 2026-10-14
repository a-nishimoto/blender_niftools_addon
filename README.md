# Blender Niftools Add-on

Import and export NetImmerse/Gamebryo files, including `.nif` and `.kf`, in Blender.
This fork packages NifTools as a Blender **extension**, with its Python dependencies
included in the installation ZIP.

## Requirements

- **Blender 5.0 or newer**, as specified in
  [`blender_manifest.toml`](io_scene_niftools/blender_manifest.toml).
  Older upstream instructions mentioning Blender 2.83 do not apply to this checkout.
- To build from source: Git, **Python 3.11 or newer with pip**, and Docker with
  **Docker Compose v2** (`docker compose`). Start Docker Desktop, or the Docker
  daemon on Linux, before building.
- macOS/Linux builds also need Bash and `zip`; Windows builds use Command Prompt
  and PowerShell.
- Internet access during the build: the scripts download PyFFI, build tools, a
  Docker base image, and the pinned code-generation repositories.

You do **not** need to build Blender or install Python dependencies into Blender
manually. The host Python builds the package; Docker generates the NIF library;
Blender installs the bundled dependency wheels.

## Build your fork

If you already cloned your fork, open a terminal in its repository root and skip
the clone command. Otherwise, replace `YOUR_GITHUB_USERNAME` below:

```sh
git clone https://github.com/YOUR_GITHUB_USERNAME/blender_niftools_addon.git
cd blender_niftools_addon
```

The current checkout has no Git submodules. The build fetches its code-generation
dependencies using the revisions pinned in [`install/Dockerfile`](install/Dockerfile).
The old instructions to generate a module manually are unnecessary for this build.

### macOS or Linux

Run these commands from the repository root. A virtual environment gives the
scripts the `python` command they expect, even on systems that only provide `python3`:

```sh
python3 --version
python3 -m venv .venv
source .venv/bin/activate
python -m pip --version
docker compose version
docker info

# The container runs this script directly; this checkout stores it without
# executable permission, so set that permission before building.
chmod +x install/generate.sh
bash install/makezip.sh
```

Use Python 3.11 or newer when creating the environment (for example,
`python3.11 -m venv .venv` if your default `python3` is older).

The installation ZIP is written to:

```text
install/temp/blender_niftools_addon-v<VERSION>-<YYYY-MM-DD>-<COMMIT>.zip
```

### Windows

Open **Command Prompt**, change to the repository root, and run:

```bat
python --version
python -m venv .venv
call .venv\Scripts\activate.bat
python -m pip --version
docker compose version
docker info
call install\makezip.bat
```

Confirm that `python --version` reports 3.11 or newer before creating the
environment. If you select Python through the Windows launcher, you can use
`py -3.11 -m venv .venv` instead.

The Docker container also needs an executable `generate.sh`. If the build fails
with `permission denied` for that script, open **Git Bash** in the repository root,
run `chmod +x install/generate.sh`, then rerun the build in Command Prompt.

The Windows ZIP is written directly to `install`, rather than `install/temp`:

```text
install/blender_niftools_addon-v<VERSION>-<YYYY-MM-DD>-<COMMIT>.zip
```

### What the build does

The script copies the add-on into a temporary directory, downloads the PyFFI
wheel, generates `nifgen` in Docker, packages it as a wheel, and creates the
extension ZIP. Its root contains `blender_manifest.toml`, `__init__.py`, and
`wheels/`. The first build can take longer because Docker must download and build
its image. Each build clears `install/temp`; keep files you want to retain elsewhere.

## Install the ZIP in Blender

1. Open Blender **5.0 or newer**.
2. Open **Edit → Preferences → Get Extensions**.
3. Open the menu in the upper-right corner and choose **Install from Disk…**.
4. Select the ZIP produced by the build above, then confirm installation.
   Keep the ZIP intact: do not select GitHub's **Code → Download ZIP** source
   archive, which lacks the generated library and bundled dependencies.
5. In **Preferences → Add-ons**, search for `NifTools` and enable
   **NifTools (Unofficial)** if it is not already enabled.
6. Save Preferences if automatic preference saving is disabled.
7. Check **File → Import** and **File → Export** for the NIF/KF entries.

Blender documents this workflow in its
[extension installation manual](https://docs.blender.org/manual/en/5.0/editors/preferences/extensions.html).
If you install a prebuilt ZIP instead, it must be an extension build compatible
with your Blender version. Upstream releases may target a different version or
use the older add-on format.

To test subsequent changes, rebuild the ZIP and install the new ZIP through the
same dialog. Restart Blender after updating to reload Python modules. If Blender
reports that the extension is already installed and does not offer replacement,
uninstall the existing NifTools extension in Preferences, then install the new ZIP.

## Troubleshooting

| Problem | What to check |
| --- | --- |
| `python: command not found` or `No module named tomllib` | Activate the virtual environment and check `python --version`. The build scripts require Python 3.11+. |
| `No module named pip` | Use a Python installation with pip/venv support, then recreate and activate the virtual environment. On Linux, your distribution may package venv separately. |
| Docker cannot connect, or `docker compose` is unavailable | Start Docker and confirm both `docker info` and `docker compose version` succeed in the same terminal. |
| Container fails with `./generate.sh: permission denied` | Set executable permission on `install/generate.sh` as described above and rebuild. |
| Dependency downloads fail | Check the first failing pip or Docker command in the terminal output and verify access to PyPI, GitHub, and the container registry. |
| Blender rejects the ZIP or cannot import `nifgen`/`pyffi` | Use the completed build ZIP, with the manifest at its root and both wheels in `wheels/`. Do not install a source archive or copy only `io_scene_niftools` from the checkout. |
| Manifest lists a wheel that was not built | If you changed the add-on version, also update the `nifgen` wheel filename in `blender_manifest.toml` to match, then rebuild. |
| Windows cannot overwrite the ZIP | Close any archive viewer using it, then rebuild. |

Prefer installing through Blender's UI over the direct `install.sh`/`install.bat`
helpers. In this checkout, `install.bat` looks for a ZIP name without the `v`
prefix that `makezip.bat` produces, and the shell helper removes the existing
installation before building its replacement.

## Further information

- [Online project documentation](https://blender-niftools-addon.readthedocs.io)
  (some setup pages describe older upstream versions).
- [Changelog](CHANGELOG.rst), [contributors](AUTHORS.rst), and [license](LICENSE.rst).
- [Contribution guidelines](CONTRIBUTING.rst).
- [Upstream releases](https://github.com/niftools/blender_niftools_addon/releases)
  and [issue tracker](https://github.com/niftools/blender_niftools_addon/issues).
  For fork-specific problems, use your fork's issue tracker and include your
  Blender version, OS, commit, and build output or Blender error.
