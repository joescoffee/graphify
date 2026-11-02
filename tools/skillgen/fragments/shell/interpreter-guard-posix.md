```bash
# Security: graphify-out/ may come from a cloned repo; trust only an absolute path outside it.
_GFY_PY=$(cat graphify-out/.graphify_python 2>/dev/null | tr -d '[:space:]')
case "$_GFY_PY" in *[!a-zA-Z0-9/_.@-]*|*/../*|"$(pwd -P)"/*|"$(pwd)"/*) _GFY_PY="" ;; /*) ;; *) _GFY_PY="" ;; esac
if [ -n "$_GFY_PY" ] && { [ ! -f "$_GFY_PY" ] || [ ! -x "$_GFY_PY" ] || ! "$_GFY_PY" -c "import graphify" 2>/dev/null; }; then _GFY_PY=""; fi
if [ -z "$_GFY_PY" ]; then
    GRAPHIFY_BIN=$(which graphify 2>/dev/null)
    if [ -n "$GRAPHIFY_BIN" ]; then
        PYTHON=$(head -1 "$GRAPHIFY_BIN" | tr -d '#!')
        case "$PYTHON" in *[!a-zA-Z0-9/_.@-]*) PYTHON="python3" ;; esac
    else
        PYTHON="python3"
    fi
    mkdir -p graphify-out
    "$PYTHON" -c "import sys; open('graphify-out/.graphify_python', 'w', encoding='utf-8').write(sys.executable)"
fi
```
