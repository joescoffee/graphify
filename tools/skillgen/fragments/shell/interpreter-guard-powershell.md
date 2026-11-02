```powershell
# Security: graphify-out\ may come from a cloned repo, so never run a recorded
# interpreter blindly. Keep it only if it is an absolute path outside this
# directory (no '..') to an existing file; otherwise re-detect.
$_gfyPy = if (Test-Path graphify-out\.graphify_python) { (Get-Content -Raw graphify-out\.graphify_python).Trim() } else { "" }
$_gfyRoot = (Get-Location).Path
if ($_gfyPy -and ((-not [System.IO.Path]::IsPathRooted($_gfyPy)) -or $_gfyPy.Contains("..") -or $_gfyPy.StartsWith($_gfyRoot, [System.StringComparison]::OrdinalIgnoreCase) -or (-not (Test-Path -PathType Leaf $_gfyPy)))) { $_gfyPy = "" }
if (-not $_gfyPy) {
    $GRAPHIFY_PYTHON = $null
    $graphifyCmd = Get-Command graphify -ErrorAction SilentlyContinue
    if ($graphifyCmd) {
        # The interpreter that owns the graphify entry point sits next to it
        # (<env>\Scripts\python.exe for uv tool, pipx, and venv installs).
        $py = Join-Path (Split-Path $graphifyCmd.Source) "python.exe"
        if (Test-Path $py) { $GRAPHIFY_PYTHON = $py }
    }
    if (-not $GRAPHIFY_PYTHON) { $GRAPHIFY_PYTHON = "python" }
    New-Item -ItemType Directory -Force -Path graphify-out | Out-Null
    & $GRAPHIFY_PYTHON -c "import sys; open('graphify-out/.graphify_python', 'w', encoding='utf-8').write(sys.executable)"
}
```
