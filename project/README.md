# Sample project — Docker Sandboxes lab

A tiny Python calculator used throughout the lab as fodder for sandboxed
agents. The `add` function has a deliberate bug — `test_add` will fail
until you fix it (you'll do that in the branch-mode section).

## Files

- `calculator.py` — four arithmetic functions, one with a bug
- `test_calculator.py` — pytest suite
- `requirements.txt` — just `pytest`

## Run the tests (inside a sandbox)

```bash
sbx exec -- sh -c "pip install --quiet -r requirements.txt && pytest -v"
```
