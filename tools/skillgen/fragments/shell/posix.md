```bash
# Detect the Python interpreter that already has graphify (uv tool, pipx, venv,
# system installs). Security: this step never installs or upgrades anything and
# never runs `uv tool run --from graphifyy` (that downloads the latest release
# from PyPI unattended). If graphify is not importable, stop and let the user
# install a pinned version themselves.
PYTHON=""
GRAPHIFY_BIN=$(which graphify 2>/dev/null)
# 1. Read shebang from the graphify launcher (uv tool, pipx and pip installs)
if [ -n "$GRAPHIFY_BIN" ]; then
    _SHEBANG=$(head -1 "$GRAPHIFY_BIN" | tr -d '#!')
    case "$_SHEBANG" in
        *[!a-zA-Z0-9/_.@-]*) ;;
        *) "$_SHEBANG" -c "import graphify" 2>/dev/null && PYTHON="$_SHEBANG" ;;
    esac
fi
# 2. Fall back to python3
if [ -z "$PYTHON" ]; then PYTHON="python3"; fi
if ! "$PYTHON" -c "import graphify" 2>/dev/null; then
    echo "graphify is not installed for $PYTHON. Nothing was installed automatically."
    echo "Install a pinned version yourself, then re-run /graphify, e.g.:"
    echo "    uv tool install graphifyy==0.9.100    # or: pipx install graphifyy==0.9.100"
    exit 1
fi
# Write interpreter path for all subsequent steps (persists across invocations)
mkdir -p graphify-out
"$PYTHON" -c "import sys; open('graphify-out/.graphify_python', 'w', encoding='utf-8').write(sys.executable)"
# Save scan root so `graphify update` (no args) knows where to look next time.
# The scan path is passed through a quoted heredoc, never substituted into the
# command line itself: a bare `cd <path>` (or an unquoted heredoc, which
# still expands $()/backticks in its body) would let a malicious path execute
# as shell code the moment this line runs. The raw path is also saved to
# graphify-out/.graphify_input, and every later Python block reads it from
# there, so it is never substituted into Python source either.
"$PYTHON" -c "import os, sys; raw = sys.stdin.readline().rstrip('\n'); open('graphify-out/.graphify_input', 'w', encoding='utf-8').write(raw); out_path = os.path.abspath('graphify-out/.graphify_root'); os.chdir(raw); open(out_path, 'w', encoding='utf-8').write(os.getcwd())" <<'GRAPHIFY_ROOT_EOF'
INPUT_PATH
GRAPHIFY_ROOT_EOF
```

If the import succeeds, print nothing and move straight to Step 2. If it fails, stop: tell the user to install a pinned version themselves (for example `uv tool install graphifyy==0.9.100`) and do not install or upgrade graphify yourself.

**In every subsequent bash block, replace `python3` with `$(cat graphify-out/.graphify_python)` to use the correct interpreter.** Step 1 rewrites that file on every run from the interpreter it just verified; never use a copy that arrived with a cloned repository without running Step 1 (or the interpreter guard below) first.
