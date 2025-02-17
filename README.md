# `tint`: 🌈 **slog.Handler** that writes tinted logs

[![Go Reference](https://pkg.go.dev/badge/github.com/topi314/tint.svg)](https://pkg.go.dev/github.com/topi314/tint#section-documentation)
[![Go Report Card](https://goreportcard.com/badge/github.com/topi314/tint)](https://goreportcard.com/report/github.com/topi314/tint)

<picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://github.com/lmittmann/tint/assets/3458786/3d42f8d5-8bdf-40db-a16a-1939c88689cb">
    <source media="(prefers-color-scheme: light)" srcset="https://github.com/lmittmann/tint/assets/3458786/3d42f8d5-8bdf-40db-a16a-1939c88689cb">
    <img src="https://github.com/lmittmann/tint/assets/3458786/3d42f8d5-8bdf-40db-a16a-1939c88689cb">
</picture>
<br>
<br>

> [!NOTE]
> This is a fork of https://github.com/lmittmann/tint with custom colors

Package `tint` implements a zero-dependency [`slog.Handler`](https://pkg.go.dev/log/slog#Handler)
that writes tinted (colorized) logs. Its output format is inspired by the `zerolog.ConsoleWriter` and
[`slog.TextHandler`](https://pkg.go.dev/log/slog#TextHandler).

The output format can be customized using [`Options`](https://pkg.go.dev/github.com/topi314/tint#Options)
which is a drop-in replacement for [`slog.HandlerOptions`](https://pkg.go.dev/log/slog#HandlerOptions).

```
go get github.com/topi314/tint
```

## Usage

```go
w := os.Stderr

// create a new logger
logger := slog.New(tint.NewHandler(w, nil))

// set global logger with custom options
slog.SetDefault(slog.New(
    tint.NewHandler(w, &tint.Options{
        Level:      slog.LevelDebug,
        TimeFormat: time.Kitchen,
    }),
))
```

### Customize Colors

Colors can be customized using the `Options.LevelColors` and `Options.Colors` attributes.
See [`tint.Options`](https://pkg.go.dev/github.com/topi314/tint#Options) for details.

```go
// ANSI escape codes: https://en.wikipedia.org/wiki/ANSI_escape_code
const (
    ansiFaint            = "\033[2m"
    ansiBrightRed        = "\033[91m"
    ansiBrightRedFaint   = "\033[91;2m"
    ansiBrightGreen      = "\033[92m"
    ansiBrightYellow     = "\033[93m"
    ansiBrightYellowBold = "\033[93;1m"
    ansiBrightBlueBold   = "\033[94;1m"
    ansiBrightMagenta    = "\033[95m"
)

// create a new logger with custom colors
w := os.Stderr
logger := slog.New(
    tint.NewHandler(w, &tint.Options{
        LevelColors: map[slog.Level]string{
            slog.LevelDebug: ansiBrightMagenta,
            slog.LevelInfo:  ansiBrightGreen,
            slog.LevelWarn:  ansiBrightYellow,
            slog.LevelError: ansiBrightRed,
        },
		Colors: map[tint.Kind]string{
            tint.KindTime:            ansiBrightYellowBold,
            tint.KindSourceFile:      ansiFaint,
            tint.KindSourceSeparator: ansiFaint,
            tint.KindSourceLine:      ansiFaint,
            tint.KindMessage:         ansiBrightBlueBold,
            tint.KindKey:             ansiFaint,
            tint.KindSeparator:       ansiFaint,
            tint.KindValue:           ansiBrightBlueBold,
            tint.KindErrorKey:        ansiBrightRedFaint,
            tint.KindErrorSeparator:  ansiBrightRedFaint,
            tint.KindErrorValue:      ansiBrightRed,
		},
    }),
)
```


### Customize Attributes

`ReplaceAttr` can be used to alter or drop attributes. If set, it is called on
each non-group attribute before it is logged. See [`slog.HandlerOptions`](https://pkg.go.dev/log/slog#HandlerOptions)
for details.

```go
// create a new logger that doesn't write the time
w := os.Stderr
logger := slog.New(
    tint.NewHandler(w, &tint.Options{
        ReplaceAttr: func(groups []string, a slog.Attr) slog.Attr {
            if a.Key == slog.TimeKey && len(groups) == 0 {
                return slog.Attr{}
            }
            return a
        },
    }),
)
```

### Automatically Enable Colors

Colors are enabled by default and can be disabled using the `Options.NoColor`
attribute. To automatically enable colors based on the terminal capabilities,
use e.g. the [`go-isatty`](https://github.com/mattn/go-isatty) package.

```go
w := os.Stderr
logger := slog.New(
    tint.NewHandler(w, &tint.Options{
        NoColor: !isatty.IsTerminal(w.Fd()),
    }),
)
```

### Windows Support

Color support on Windows can be added by using e.g. the
[`go-colorable`](https://github.com/mattn/go-colorable) package.

```go
w := os.Stderr
logger := slog.New(
    tint.NewHandler(colorable.NewColorable(w), nil),
)
```
