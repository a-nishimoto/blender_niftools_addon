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

## Build instructions

Open a terminal in the root of the local repository folder and follow the
instructions for your operating system below.

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
4. Select the completed extension ZIP, then confirm installation.
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
| Container fails with `./generate.sh: permission denied` | Run `chmod +x install/generate.sh` in Bash or Git Bash from the repository root, then rebuild. |
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

## Dependencies and build downloads

Cloning this repository downloads no dependencies: the current checkout has no
Git submodules. Running `install/makezip.sh` or `install/makezip.bat` downloads
build dependencies and generates the library bundled with the extension.

### Python packages and Docker image

| Item | What is fetched or built |
| --- | --- |
| Bundled Python dependency | **PyFFI `2.2.4.dev3`**, downloaded as a wheel from the configured pip index. `--no-deps` prevents fetching its dependencies. |
| Generated Python dependency | **nifgen `1.1.0`**, generated in Docker and packaged locally as a wheel. Its version follows the add-on manifest. |
| Host Python build tool | **setuptools `>=61.0`**, installed into pip's isolated build environment for the nifgen wheel. |
| Docker base image | **`python:3.10.8`**, normally pulled from Docker Hub unless cached. |
| Container build tools | **pip** and **setuptools** are upgraded without version pins. |
| Container Python dependencies | The pinned cobra-tools repository's **`requirements.txt`**, plus any transitive dependencies resolved by pip. |

See [`install/makezip.sh`](install/makezip.sh),
[`install/makezip.bat`](install/makezip.bat),
[`install/build_nifgen_wheel.py`](install/build_nifgen_wheel.py), and
[`install/Dockerfile`](install/Dockerfile).

### Code-generation repositories

The Docker build performs ordinary Git clones, then checks out these pinned
commits:

| Repository | Commit | Purpose |
| --- | --- | --- |
| [Candoran2/cobra-tools](https://github.com/Candoran2/cobra-tools) | `99cbdf7fcb7709fcf0e4dc363d17a50ef0410ad7` | Code generator |
| [DuncanWasHere/new-pyffi](https://github.com/DuncanWasHere/new-pyffi) | `f188986d1d72a31ffc6a981a57f1337e35030d3a` | NIF definitions, spells, and utilities |
| [Candoran2/nifxml](https://github.com/Candoran2/nifxml) | `9914a2b4984b4909cbc55512fa8162852b211565` | `nif.xml` format specification |

[`install/docker-compose.yml`](install/docker-compose.yml) builds one local
`codegen` service and mounts the generated output directory. It does not reference
additional prebuilt images.

### Tools required on the host

Git, **Python 3.11 or newer with pip**, Docker, and **Docker Compose v2** must
already be installed. macOS/Linux packaging also uses Bash and `zip`; Windows
packaging uses Command Prompt and PowerShell. Blender itself is not downloaded
or built by these scripts.

### Optional test, documentation, and CI dependencies

- **Integration tests:** [`testframework/install_deps.sh`](testframework/install_deps.sh)
  installs Sphinx, nose, PyFFI `2.2.4.dev4`, and
  `pydevd-pycharm~=201.6668.60`. The Windows test installer installs Sphinx, nose,
  and the same debugger constraint, but omits an active PyFFI installation.
- **Documentation:** [`docs/requirements.txt`](docs/requirements.txt) specifies
  `sphinx>=1.5.6` and `niftools_sphinx_theme>=0.3.3`.
- **GitHub CI:** The release workflow uses `actions/checkout@v4` and
  `actions/setup-python@v5` with Python 3.11. A separate PR workflow uses
  `technote-space/pr-commit-body-action@v1`. Both workflows use `ubuntu-latest`.

The exact contents of cobra-tools' pinned `requirements.txt` have not been
verified; attempts to retrieve that file during this dependency audit failed.
The repository revisions and bundled PyFFI version are pinned, but the complete
Python build environment is not locked. The Docker base image is pinned by tag,
not by digest.
