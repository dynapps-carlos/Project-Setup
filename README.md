# Dynapps Project Initialization Manual

This guide outlines the step-by-step process to initialize a project following Dynapps development conventions.

---

## 1. Clone the Customer Repository

```bash
git clone git@github.com:dynapps-sa/3epis.git 3epis15
cd 3epis15/
git checkout -b 15.0-dynapps
```

## 2. Generate Project Structure with Copier

```bash
copier copy git@gitlab.dynapps.be:verticals/skeleton-template.git .
```

### Answer the prompts as follows (using keyboard cursor):

* **Odoo version**: `V15`
* **Organization name**: `dynapps`
* **Repository to initialize**: `Customer Repo`
* **CI service**: `GitHub CI`
* **Use ruff and ruff-format**: `No`
* **Fix website key in manifest with pre-commit**: `No`
* **Fix README with pre-commit**: `No`

## 3. Project Setup

### Open the project in PyCharm.

### Move and Configure Modules

* Move non-submodule OCA modules to `parts/oca_pull`.
* Add those modules to the `depends` of `xx_base`.
* Move Odoo Apps modules to `parts/odoo_apps/`.
* Add the following to `.gitignore`:

```ini
!/parts/odoo_apps/
```

* Add `odoo_apps` to `dyn-build.yaml` before the `oca_pull` section:

```yaml
- repo: parts/odoo_apps
  source: local
```

* Add Odoo Apps modules to the `depends` of `xx_base`.
* Add `parts/odoo_apps` and `parts/oca_pull` to the ignore list of `.pre-commit-config.yaml`:

```yaml
# OCA ported modules to ignore
parts/oca_pull|
parts/odoo_apps
```

* Move custom modules to `custom/`.
* Add custom modules to `depends` of `xx_all`.

### Clean xx\_base

* Remove unnecessary content from `xx_base`.

### Git Submodules

* Empty the `.gitmodules` file.
* Remove any existing OCA folder.
* Add `oca` to `.gitignore`:

```ini
!/parts/oca/
```

* Add the submodule again:

```bash
git submodule add https://github.com/OCA/partner-contact parts/oca/partner-contact
cd parts/oca/partner-contact
git checkout <commit_hash>
```

* Add the submodule to `dyn-build.yaml`:

```yaml
- repo: parts/oca/partner-contact.git
  source: local
```

## 4. Cleanup

* Clean up `dyn-build.yaml`:

  * Remove Dynapps module repos.
  * Remove other OCA module repos.
  * Remove the `config/` folder and reference.
* In `.pre-commit-config.yaml`, remove:

```yaml
- id: oca-fix-manifest-website
  args: ["https://www.dynapps.eu"]
```

* In `requirements-dev.txt`, add:

```txt
cython==3.0.11
```

## 5. Commit Initial State

```bash
git add .
git commit -m "[IMP] Dynapps way of working" --no-verify
pre-commit run -a
```

## 6. Build and Test

```bash
dyn-tools build update --build-profile dev-no-update
```

## 7. Python Environment Setup (using `uv`)

```bash
uv python pin 3.10.0
uv venv 3epis15
source 3epis15/bin/activate
uv pip install -r requirements-dev.txt
```

## 8. Run Odoo

```bash
./parts/odoo/odoo/odoo-bin -c odoo.cfg -d 3epis15 -i xx_all
```

## 9. Cursor AI configuration files

Launch.json:

```json
{
    "version": "0.2.0",
    "configurations": [
      {
        "name": "Odoo (dev)",
        "type": "debugpy",
        "request": "launch",
  
        // odoo-bin
        "program": "${workspaceFolder}/parts/odoo/odoo/odoo-bin",
  
        // Odoo parameters
        "args": [
          "-c", "odoo.cfg",
          "-d", "dootix17_internal",
          "-u", "dynapps_developments",
          "--limit-memory-hard=0"
        ],
  
        "cwd": "${workspaceFolder}",
        "console": "integratedTerminal",
        "stopOnEntry": false,
        "justMyCode": false
      }
    ]
  }
  
```

Settings.json:

```json
{
    // Equivalent to "Set as Source Root" for Odoo
    "python.analysis.extraPaths": [
      "${workspaceFolder}/parts/odoo/odoo",
      "${workspaceFolder}/parts/odoo/enterprise"
    ],
    "python.languageServer": "None",
  }
  
```

---

## 🛠 Troubleshooting

### Flake8 Errors

If `flake8` fails due to plugin compatibility, apply the following fix in `.pre-commit-config.yaml`:

```yaml
- repo: https://github.com/PyCQA/flake8
  rev: 7.0.0
  hooks:
    - id: flake8
      name: flake8
      additional_dependencies: ["flake8-bugbear==24.2.6"]
```

### Gevent Compatibility for Python 3.10+

If you experience issues with `gevent` when using Python 3.10+, you can patch the `requirements.txt` file.

Modify `dyn-build.yaml`:

```yaml
- branch: 15.0
  patches:
    - odoo/requirements.txt.patch
  repo: odoo/odoo.git
  source: dynapps
```

Create file `patches/odoo/requirements.txt.patch`:

```diff
Index: requirements.txt
===================================================================
@@ -9,11 +9,11 @@
 freezegun==0.3.15; python_version >= '3.8'
 gevent==1.5.0 ; sys_platform != 'win32' and python_version == '3.7'
 gevent==20.9.0 ; sys_platform != 'win32' and python_version > '3.7' and python_version <= '3.9'
-gevent==21.8.0 ; sys_platform != 'win32' and python_version > '3.9' and python_version < '3.12' # (Jammy)
+gevent==22.10.2 ; sys_platform != 'win32' and python_version > '3.9' and python_version < '3.12' # (Jammy)
 gevent==24.2.1 ; sys_platform != 'win32' and python_version >= '3.12'  # (Noble)
 greenlet==0.4.15 ; sys_platform != 'win32' and python_version == '3.7'
 greenlet==0.4.17 ; sys_platform != 'win32' and python_version > '3.7' and python_version <= '3.9'
-greenlet==1.1.2 ; sys_platform != 'win32' and python_version > '3.9' and python_version < '3.12' # (Jammy)
+greenlet==2.0.2 ; sys_platform != 'win32' and python_version > '3.9' and python_version < '3.12' # (Jammy)
 greenlet==3.0.3 ; sys_platform != 'win32' and python_version >= '3.12'  # (Noble)
 idna==2.8
 Jinja2==2.11.3 ; python_version < '3.12'
```

> Alternatively, you can manually replace the affected lines in `/odoo/requirements.txt` to upgrade `gevent` and `greenlet` to higher compatible versions if you're not using patches.
