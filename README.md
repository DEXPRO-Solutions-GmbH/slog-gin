
# slog: Gin middleware

[![tag](https://img.shields.io/github/tag/samber/slog-gin.svg)](https://github.com/samber/slog-gin/releases)
![Go Version](https://img.shields.io/badge/Go-%3E%3D%201.20.3-%23007d9c)
[![GoDoc](https://godoc.org/github.com/samber/slog-gin?status.svg)](https://pkg.go.dev/github.com/samber/slog-gin)
![Build Status](https://github.com/samber/slog-gin/actions/workflows/test.yml/badge.svg)
[![Go report](https://goreportcard.com/badge/github.com/samber/slog-gin)](https://goreportcard.com/report/github.com/samber/slog-gin)
[![Coverage](https://img.shields.io/codecov/c/github/samber/slog-gin)](https://codecov.io/gh/samber/slog-gin)
[![Contributors](https://img.shields.io/github/contributors/samber/slog-gin)](https://github.com/samber/slog-gin/graphs/contributors)
[![License](https://img.shields.io/github/license/samber/slog-gin)](./LICENSE)

[Gin](https://github.com/gin-gonic/gin) middleware to log http requests using [slog](https://pkg.go.dev/golang.org/x/exp/slog).

**See also:**

- [slog-multi](https://github.com/samber/slog-multi): workflows of `slog` handlers (pipeline, fanout, ...)
- [slog-formatter](https://github.com/samber/slog-formatter): `slog` attribute formatting
- [slog-datadog](https://github.com/samber/slog-datadog): A `slog` handler for `Datadog`
- [slog-logstash](https://github.com/samber/slog-logstash): A `slog` handler for `Logstash`
- [slog-slack](https://github.com/samber/slog-slack): A `slog` handler for `Slack`
- [slog-loki](https://github.com/samber/slog-loki): A `slog` handler for `Loki`
- [slog-sentry](https://github.com/samber/slog-sentry): A `slog` handler for `Sentry`
- [slog-fluentd](https://github.com/samber/slog-fluentd): A `slog` handler for `Fluentd`
- [slog-syslog](https://github.com/samber/slog-syslog): A `slog` handler for `Syslog`
- [slog-graylog](https://github.com/samber/slog-graylog): A `slog` handler for `Graylog`

## 🚀 Install

```sh
go get github.com/samber/slog-gin
```

**Compatibility**: go >= 1.20.3

This library is v0 and follows SemVer strictly. On `slog` final release (go 1.21), this library will go v1.

No breaking changes will be made to exported APIs before v1.0.0.

## 💡 Usage

### Minimal

```go
import (
	"net/http"
	"os"
	"time"

	"github.com/gin-gonic/gin"
	sloggin "github.com/samber/slog-gin"
	"golang.org/x/exp/slog"
)

// Create a slog logger, which:
//   - Logs to stdout.
logger := slog.New(slog.NewTextHandler(os.Stdout))

router := gin.New()

// Add the sloggin middleware to all routes.
// The middleware will log all requests attributes.
router.Use(sloggin.New(logger))

// Example pong request.
router.GET("/pong", func(c *gin.Context) {
    c.String(http.StatusOK, "pong")
})

router.Run(":1234")

// output:
// time=2023-04-10T14:00:0.000000+02:00 level=INFO msg="Incoming request" status=200 method=GET path=/pong ip=127.0.0.1 latency=25.5µs user-agent=curl/7.77.0 time=2023-04-10T14:00:00.000+02:00
```

### Using custom time formatters

```go
import (
	"net/http"
	"os"
	"time"

	"github.com/gin-gonic/gin"
	sloggin "github.com/samber/slog-gin"
	"golang.org/x/exp/slog"
)

// Create a slog logger, which:
//   - Logs to stdout.
//   - RFC3339 with UTC time format.
logger := slog.New(
    slogformatter.NewFormatterHandler(
        slogformatter.TimezoneConverter(time.UTC),
        slogformatter.TimeFormatter(time.DateTime, nil),
    )(
        slog.NewTextHandler(os.Stdout),
    ),
)

router := gin.New()

// Add the sloggin middleware to all routes.
// The middleware will log all requests attributes.
router.Use(sloggin.New(logger))

// Example pong request.
router.GET("/pong", func(c *gin.Context) {
    c.String(http.StatusOK, "pong")
})

router.Run(":1234")

// output:
// time="2023-04-10 14:00:00" level=INFO msg="Incoming request" status=200 method=GET path=/pong ip=127.0.0.1 latency=25.5µs user-agent=curl/7.77.0 time="2023-04-10 14:00:00"
```

### Using custom logger sub-group

```go
import (
	"net/http"
	"os"
	"time"

	"github.com/gin-gonic/gin"
	sloggin "github.com/samber/slog-gin"
	"golang.org/x/exp/slog"
)

// Create a slog logger, which:
//   - Logs to stdout.
logger := slog.New(slog.NewTextHandler(os.Stdout))

router := gin.New()

// Add the sloggin middleware to all routes.
// The middleware will log all requests attributes under a "http" group.
router.Use(sloggin.New(logger.WithGroup("http")))

// Example pong request.
router.GET("/pong", func(c *gin.Context) {
    c.String(http.StatusOK, "pong")
})

router.Run(":1234")

// output:
// time=2023-04-10T14:00:0.000000+02:00 level=INFO msg="Incoming request" http.status=200 http.method=GET http.path=/pong http.ip=127.0.0.1 http.latency=20.125µs http.user-agent=curl/7.77.0 time=2023-04-10T14:00:00.000+02:00
```

### Add logger to a single route

```go
import (
	"net/http"
	"os"
	"time"

	"github.com/gin-gonic/gin"
	sloggin "github.com/samber/slog-gin"
	"golang.org/x/exp/slog"
)

// Create a slog logger, which:
//   - Logs to stdout.
logger := slog.New(slog.NewTextHandler(os.Stdout))

router := gin.New()

// Example pong request.
// Add the sloggin middleware to a single routes.
router.GET("/pong", sloggin.New(logger), func(c *gin.Context) {
    c.String(http.StatusOK, "pong")
})

router.Run(":1234")

// output:
// time="2023-04-10 14:00:00" level=INFO msg="Incoming request" status=200 method=GET path=/pong ip=127.0.0.1 latency=25.5µs user-agent=curl/7.77.0 time="2023-04-10 14:00:00"
```

### Adding custom attributes

```go
import (
	"net/http"
	"os"
	"time"

	"github.com/gin-gonic/gin"
	sloggin "github.com/samber/slog-gin"
	"golang.org/x/exp/slog"
)

// Create a slog logger, which:
//   - Logs to stdout.
logger := slog.New(slog.NewTextHandler(os.Stdout)).
    With("environment", "production").
    With("server", "gin/1.9.0").
    With("server_start_time", time.Now()).
    With("gin_mode", gin.EnvGinMode)

router := gin.New()

// Add the sloggin middleware to all routes.
// The middleware will log all requests attributes.
router.Use(sloggin.New(logger))

// Example pong request.
router.GET("/pong", func(c *gin.Context) {
    c.String(http.StatusOK, "pong")
})

router.Run(":1234")

// output:
// time=2023-04-10T14:00:0.000000+02:00 level=INFO msg="Incoming request" environment=production server=gin/1.9.0 gin_mode=release server_start_time=2023-04-10T10:00:00.000+02:00 status=200 method=GET path=/pong ip=127.0.0.1 latency=25.5µs user-agent=curl/7.77.0 time=2023-04-10T14:00:00.000+02:00
```

### JSON output

```go
import (
	"net/http"
	"os"
	"time"

	"github.com/gin-gonic/gin"
	sloggin "github.com/samber/slog-gin"
	"golang.org/x/exp/slog"
)

// Create a slog logger, which:
//   - Logs to stdout.
logger := slog.New(slog.NewJSONHandler(os.Stdout))

router := gin.New()

// Add the sloggin middleware to all routes.
// The middleware will log all requests attributes.
router.Use(sloggin.New(logger))

// Example pong request.
router.GET("/pong", func(c *gin.Context) {
    c.String(http.StatusOK, "pong")
})

router.Run(":1234")

// output:
// {"time":"2023-04-10T14:00:0.000000+02:00","level":"INFO","msg":"Incoming request","gin_mode":"GIN_MODE","status":200,"method":"GET","path":"/pong","ip":"127.0.0.1","latency":15542,"user-agent":"curl/7.77.0","time":"2023-04-10T14:00:0.000000+02:00"}
```

## 🤝 Contributing

- Ping me on twitter [@samuelberthe](https://twitter.com/samuelberthe) (DMs, mentions, whatever :))
- Fork the [project](https://github.com/samber/slog-gin)
- Fix [open issues](https://github.com/samber/slog-gin/issues) or request new features

Don't hesitate ;)

```bash
# Install some dev dependencies
make tools

# Run tests
make test
# or
make watch-test
```

## 👤 Contributors

![Contributors](https://contrib.rocks/image?repo=samber/slog-gin)

## 💫 Show your support

Give a ⭐️ if this project helped you!

[![GitHub Sponsors](https://img.shields.io/github/sponsors/samber?style=for-the-badge)](https://github.com/sponsors/samber)

## 📝 License

Copyright © 2023 [Samuel Berthe](https://github.com/samber).

This project is [MIT](./LICENSE) licensed.
