# Border Animation

A traveling spark animation on the table border indicates the refresh cycle progress.

## How It Works

- A short gradient spark (white head fading to the border color) travels clockwise around the table border
- Uses cubic ease-in-out timing: starts slow, accelerates in the middle, decelerates at the end
- The spark grows from nothing at the start and shrinks to nothing at the end (comet effect)
- After completing a lap, the entire border holds the focus color for 250ms before the next lap begins
- The top-left corner is excluded from the spark to avoid visual overlap with the downward stroke
- Title text on the top border is detected and preserved — only border characters are redrawn
- Animation is skipped entirely during initial load (first two refresh cycles) so the UI appears instantly

## Configuration

In `~/.config/k9s/config.yaml`:

```yaml
k9s:
  animationFPS: 240     # Animation frame rate (default: 240)
  refreshRate: 2         # Data refresh interval in seconds
```

| Setting        | Default | Description                                      |
|----------------|---------|--------------------------------------------------|
| `animationFPS` | `240`   | Frames per second for the border animation        |
| `refreshRate`  | `2`     | How often data refreshes (seconds). The spark completes one lap per cycle minus a 250ms pause. |

## Architecture

| File | Role |
|------|------|
| `internal/ui/table.go` | `Draw` override and `drawProgressBorder` — builds the perimeter cell list, computes easing, renders the gradient spark |
| `internal/ui/types.go` | `Tabular` interface additions: `GetRefreshRate`, `GetLastRefreshAt`, `GetRefreshCount` |
| `internal/model/table.go` | Tracks `lastRefreshAt`, `refreshCount`, and exposes getters |
| `internal/view/browser.go` | Animation goroutine — ticks at configured FPS, gates on `refreshCount >= 2` |
| `internal/config/k9s.go` | `AnimationFPS` field, `GetAnimationTickInterval()` (converts FPS → tick duration) |
| `internal/config/types.go` | `defaultAnimationFPS` constant |
