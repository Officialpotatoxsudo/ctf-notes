# Python venv / pip Cheatsheet

Keep project dependencies isolated and reproducible.

## 1. Create and use a virtual environment

```bash
python3 -m venv .venv
source .venv/bin/activate
python -V
which python
```

Windows:

```powershell
py -m venv .venv
.venv\Scripts\Activate.ps1
```

Deactivate:

```bash
deactivate
```

## 2. Install packages

```bash
python -m pip install requests
python -m pip install "requests==2.32.3"
python -m pip install -r requirements.txt
python -m pip install -e .
```

Prefer `python -m pip` over bare `pip` so you know which interpreter is used.

## 3. Freeze dependencies

```bash
python -m pip freeze > requirements.txt
```

For libraries, separate direct dependencies from the lock file:

```text
requirements.in   # direct dependencies
requirements.txt  # fully pinned lock file
```

Tools: `pip-tools`, `uv`, `poetry`, `pdm`.

## 4. Inspect the environment

```bash
python -m pip list
python -m pip show requests
python -m pip check
python -m site
```

## 5. Uninstall and clean

```bash
python -m pip uninstall requests
python -m pip cache dir
python -m pip cache purge
```

## 6. Reproducible installs

```bash
python -m pip install --require-hashes -r requirements.txt
python -m pip install --no-deps -r requirements.txt
```

`--require-hashes` is useful for supply-chain-sensitive projects.

## 7. Common errors

**`externally-managed-environment`** (PEP 668): use a venv, `pipx`, or
`--break-system-packages` only as a last resort.

```bash
python3 -m venv .venv
source .venv/bin/activate
```

**Wrong interpreter**: check `which python` and `python -m pip -V`.

**`ModuleNotFoundError` after install**: the package went into a different environment.
Use `python -m pip install` inside the active venv.

**SSL certificate errors**: update `certifi`, or set `SSL_CERT_FILE` to the correct bundle.

## 8. pipx for CLI tools

```bash
pipx install ruff
pipx install black
pipx list
pipx upgrade-all
pipx uninstall ruff
```

Use `pipx` for standalone tools, venv for project dependencies.

## 9. uv quickstart

```bash
uv venv
source .venv/bin/activate
uv pip install requests
uv pip compile requirements.in -o requirements.txt
uv pip sync requirements.txt
```

`uv` is much faster than pip and is a drop-in for many workflows.

## 10. Project layout

```text
project/
├── .venv/
├── pyproject.toml
├── requirements.in
├── requirements.txt
├── src/
│   └── package/
└── tests/
```

Add `.venv/` and `__pycache__/` to `.gitignore`. Commit the lock file, not the venv.

## 11. Quick checklist

- [ ] venv created with the intended Python version
- [ ] dependencies pinned
- [ ] lock file committed
- [ ] `pip check` passes
- [ ] no secrets in `requirements.txt`
- [ ] CI uses the same Python version
