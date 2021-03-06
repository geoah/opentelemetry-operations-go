# OpenTelemetry Google Cloud Trace Exporter

[![Docs](https://godoc.org/github.com/GoogleCloudPlatform/opentelemetry-operations-go/exporter/trace?status.svg)](https://pkg.go.dev/github.com/GoogleCloudPlatform/opentelemetry-operations-go/exporter/trace)
[![Apache License][license-image]][license-url]

OpenTelemetry Google Cloud Trace Exporter allows the user to send collected traces and spans to Google Cloud.

[Google Cloud Trace](https://cloud.google.com/trace) is a distributed tracing backend system. It helps developers to gather timing data needed to troubleshoot latency problems in microservice & monolithic architectures. It manages both the collection and lookup of gathered trace data.

This exporter package assumes your application is [already instrumented](https://github.com/open-telemetry/opentelemetry-go/blob/master/example/http/client/client.go) with the OpenTelemetry SDK. Once you get ready to export OpenTelemetry data, you can add this exporter to your application.

## Setup

Google Cloud Trace is a managed service provided by Google Cloud Platform. The end-to-end set up guide with OpenTelemetry is available on [the official GCP docs](https://cloud.google.com/trace/docs/setup/go-ot), so this document goes through the exporter set up.

## Usage

Once you import the trace exporter package, create and install a new export pipeline, then you can start tracing. If you are running in a GCP environment, the exporter will automatically authenticate using the environment's service account. If not, you will need to follow the instruction in [Authentication](#Authentication).

```go
package main

import (
    "context"
    "log"
    "os"

    texporter "github.com/GoogleCloudPlatform/opentelemetry-operations-go/exporter/trace"

    "go.opentelemetry.io/otel/api/global"
    sdktrace "go.opentelemetry.io/otel/sdk/trace"
)

func main() {
    // Create exporter and trace provider pipeline, and register provider.
    _, flush, err := texporter.InstallNewPipeline(
        []texporter.Option {
            // optional exporter options
        },
        // This example code uses sdktrace.AlwaysSample sampler to sample all traces.
        // In a production environment or high QPS setup please use ProbabilitySampler
        // set at the desired probability.
        // Example:
        // sdktrace.WithConfig(sdktrace.Config {
        //     DefaultSampler: sdktrace.ProbabilitySampler(0.0001),
        // })
        sdktrace.WithConfig(sdktrace.Config{
            DefaultSampler: sdktrace.AlwaysSample(),
        }),
        // other optional provider options
    )
    if err != nil {
        log.Fatalf("texporter.InstallNewPipeline: %v", err)
    }
    // before ending program, wait for all enqueued spans to be exported
    defer flush()

    // Create custom span.
    tracer := global.TraceProvider().Tracer("example.com/trace")
    err = func(ctx context.Context) error {
        ctx, span := tracer.Start(ctx, "foo")
        defer span.End()

        // Do some work.

        return nil
    }(ctx)
}
```

## Authentication

The Google Cloud Trace exporter depends upon [`google.FindDefaultCredentials`](https://pkg.go.dev/golang.org/x/oauth2/google?tab=doc#FindDefaultCredentials), so the service account is automatically detected by default, but also the custom credential file (so called `service_account_key.json`) can be detected with specific conditions. Quoting from the document of `google.FindDefaultCredentials`:

* A JSON file whose path is specified by the `GOOGLE_APPLICATION_CREDENTIALS` environment variable.
* A JSON file in a location known to the gcloud command-line tool. On Windows, this is `%APPDATA%/gcloud/application_default_credentials.json`. On other systems, `$HOME/.config/gcloud/application_default_credentials.json`.

When running code locally, you may need to specify a Google Project ID in addition to `GOOGLE_APPLICATION_CREDENTIALS`. This is best done using an environment variable (e.g. `GOOGLE_CLOUD_PROJECT`) and the `WithProjectID` method, e.g.:

```go
projectID := os.Getenv("GOOGLE_CLOUD_PROJECT")
_, flush, err := texporter.InstallNewPipeline(
    []texporter.Option {
        texporter.WithProjectID(projectID),
        // other optional exporter options
    },
    ...
)
```

## Useful links

* For more information on OpenTelemetry, visit: https://opentelemetry.io/
* For more about OpenTelemetry Go, visit: https://github.com/open-telemetry/opentelemetry-go
* Learn more about Google Cloud Trace at https://cloud.google.com/trace

[license-url]: https://github.com/GoogleCloudPlatform/opentelemetry-operations-go/blob/master/LICENSE
[license-image]: https://img.shields.io/badge/license-Apache_2.0-green.svg?style=flat
