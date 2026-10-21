# Stable Diffusion Web UI: agent context

Gradio UI and FastAPI API for Stable Diffusion image generation, image-to-image,
upscaling and model tooling. This checkout is based on `82a973c`; validate against
its files rather than assuming newer Gradio, PyTorch or JavaScript conventions.

## Where changes belong
- `webui.sh` / `webui-user.bat` bootstrap the environment; `launch.py` delegates
  dependency preparation and test-server setup to `modules/launch_utils.py`.
- `webui.py` initializes the app, then builds the UI through `modules/ui.py` or
  exposes routes from `modules/api/api.py`; generation flows through
  `modules/txt2img.py`, `modules/img2img.py` and `modules/processing.py`.
- `scripts/` contains built-in `modules.scripts.Script` implementations;
  `extensions-builtin/` contains bundled extensions. User extensions live in
  `extensions/`; extension hooks are in `modules/script_callbacks.py`.
- `script.js`, `javascript/` and `style.css` are browser code, served by Python;
  `configs/` holds model configurations; `test/` holds pytest tests and fixtures.

## Commands (from the repository root)
Use a Python 3.10 virtual environment; CI tests use 3.10.6. The macOS launcher
sets platform-specific Torch wheels in `webui-macos-env.sh`.
```bash
python3.10 -m venv venv
source venv/bin/activate
python -m pip install -r requirements-test.txt ruff==0.3.3
# Apple Silicon utility-test bootstrap; no server or model weights required.
python -m pip install 'numpy<2' torch==2.3.1
python -m pytest -q test/test_torch_utils.py
ruff check .
npm install
npm run lint
```
There is no frontend build step or `npm test` / `npm run build`; npm only installs
ESLint. `npm run fix` applies ESLint fixes; do not bulk-reformat unrelated files.
For normal app setup/run: `python_cmd=python3.10 bash webui.sh --api` (downloads
runtime dependencies and may download models). Customize `webui-user.sh`, not
platform defaults. Do not run the launcher just to lint or inspect a utility.

Most tests are HTTP integration tests: bare `pytest test/` needs the live API.
For the Linux CPU/empty-checkpoint CI path, after activating the environment:
```bash
python -m pip install wait-for-it -r requirements-test.txt
TORCH_INDEX_URL=https://download.pytorch.org/whl/cpu python launch.py --skip-torch-cuda-test --exit
python launch.py --skip-prepare-environment --test-server --skip-torch-cuda-test --do-not-download-clip --no-half --disable-opt-split-attention --use-cpu all --api-server-stop
```
Leave that process running; in a second terminal with the same venv activated:
`python -m pytest -vv --verify-base-url test`. The default test URL is
`http://127.0.0.1:7860`; `configure_for_tests()` selects `test/test_files/empty.pt`.
Do not describe the checkpoint-free utility smoke test as the full API suite.

## Local conventions and boundaries
- Python functions/modules use snake_case; pytest tests live in `test/test_*.py`
  and use fixtures and `pytest.mark.parametrize`. For exception paths, use
  `pytest.raises`; follow neighboring tests rather than importing the whole UI.
- Ruff settings live in `pyproject.toml` (Python 3.9 syntax target); lint CI uses
  Ruff 0.3.3. Preserve existing line endings and quoting; no Python formatter is
  prescribed here. JS uses four spaces, semicolons and browser globals from
  `.eslintrc.js`; use `gradioApp()` and existing UI callbacks, not a React build.
- Treat cloned `repositories/`, `venv/`, `node_modules/`, model weights,
  `outputs/`, caches and coverage as dependencies/generated or user data.
  Do not commit weights, `config.json`, `ui-config.json`, `styles.csv` or personal
  launcher overrides. Preserve tracked test fixtures and bundled extensions;
  they are not disposable generated output. Do not edit user `extensions/` to
  fix core behavior. Development PRs target `dev`; the workflow rejects `master`.
