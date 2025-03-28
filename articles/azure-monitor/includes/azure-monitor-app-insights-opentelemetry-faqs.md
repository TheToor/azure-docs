---
author: AaronMaxwell
ms.author: aaronmax
ms.service: azure-monitor
ms.topic: include
ms.date: 10/13/2023
---

## Frequently asked questions

This section provides answers to common questions.

### What is OpenTelemetry?

It's a new open-source standard for observability. Learn more at [OpenTelemetry](https://opentelemetry.io/).

### Why is Microsoft Azure Monitor investing in OpenTelemetry?

Microsoft is among the largest contributors to OpenTelemetry.
          
The key value propositions of OpenTelemetry are that it's vendor-neutral and provides consistent APIs/SDKs across languages.
          
Over time, we believe OpenTelemetry will enable Azure Monitor customers to observe applications written in languages beyond our [supported languages](../app/app-insights-overview.md#supported-languages). It also expands the types of data you can collect through a rich set of [instrumentation libraries](https://opentelemetry.io/docs/concepts/components/#instrumentation-libraries). Furthermore, OpenTelemetry SDKs tend to be more performant at scale than their predecessors, the Application Insights SDKs.

Finally, OpenTelemetry aligns with Microsoft's strategy to [embrace open source](https://opensource.microsoft.com/).

### What's the status of OpenTelemetry?

See the [OpenTelemetry Spec Compliance Matrix](https://github.com/open-telemetry/opentelemetry-specification/blob/main/spec-compliance-matrix.md).

### What is the "Azure Monitor OpenTelemetry Distro"?

You can think of it as a thin wrapper that bundles together all the OpenTelemetry components for a first class experience on Azure.

### Why should I use the "Azure Monitor OpenTelemetry Distro"?

There are several advantages to using the Azure Monitor OpenTelemetry Distro over native OpenTelemetry from the community:
- Reduces enablement effort
- Supported by Microsoft
- Brings in Azure Specific features such as:
   - Preserves traces with service components using Application Insights SDKs
   - [Microsoft Entra authentication](../app/azure-ad-authentication.md)
   - [Offline Storage and Automatic Retries](../app/opentelemetry-configuration.md#offline-storage-and-automatic-retries)
   - [Statsbeat](../app/statsbeat.md)
   - [Application Insights Standard Metrics](../app/standard-metrics.md)
   - Detect resource metadata to autopopulate [Cloud Role Name](../app/app-map.md#understand-the-cloud-role-name-within-the-context-of-an-application-map) on Azure
   - [Live Metrics](../app/live-stream.md) (future)
          
In the spirit of OpenTelemetry, we designed the distro to be open and extensible. For example, you can add:
- An OTLP exporter and send to a second destination simultaneously
- Other instrumentation libraries not included in the distro

### How can I test out the Azure Monitor OpenTelemetry Distro?

Check out our enablement docs for [.NET, Java, JavaScript (Node.js), and Python](../app/opentelemetry-enable.md).

### Should I use OpenTelemetry or the Application Insights SDK?

We recommend using the OpenTelemetry Distro unless you require a feature that is only available with formal support in the Application Insights SDK.

Adopting OpenTelemetry now prevents having to migrate at a later date.

### When should I use the Azure Monitor OpenTelemetry exporter?

For ASP..NET Core, Java, Node.js, and Python, we recommend using the OpenTelemetry distro. It's one line of code to get started.

For all other .NET scenarios, we recommend using our exporter: `Azure.Monitor.OpenTelemetry.Exporter`.

### What's the current release state of features within the Azure Monitor OpenTelemetry Distro?

The following chart breaks out OpenTelemetry feature support for each language.

|Feature                                                                                                                | .NET               | Node.js            | Python             | Java               |
|-----------------------------------------------------------------------------------------------------------------------|--------------------|--------------------|--------------------|--------------------|
| [Distributed tracing](../app/distributed-tracing-telemetry-correlation.md)                                            | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: |
| [Custom metrics](../app/opentelemetry-add-modify.md#add-custom-metrics)                                               | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: |
| [Standard metrics](../app/standard-metrics.md) (accuracy currently affected by sampling)                              | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: |
| [Fixed-rate sampling](../app/sampling.md)                                                                             | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: |
| [Offline storage and automatic retries](../app/opentelemetry-configuration.md#offline-storage-and-automatic-retries)  | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: |
| [Exception reporting](../app/asp-net-exceptions.md)                                                                   | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: |
| [Logs collection](../app/asp-net-trace-logs.md)                                                                       | :white_check_mark: | :warning:          | :white_check_mark: | :white_check_mark: |
| [Custom Events](../app/usage-overview.md#custom-business-events)                                                      | :warning:          | :warning:          | :warning:          | :white_check_mark: |
| [Microsoft Entra authentication](../app/azure-ad-authentication.md)                                                   | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: |
| [Live metrics](../app/live-stream.md)                                                                                 | :x:                | :x:                | :x:                | :white_check_mark: |
| Detect Resource Context for VM/VMSS and App Service                                                                   | :white_check_mark: | :x:                | :white_check_mark: | :white_check_mark: |
| Detect Resource Context for AKS and Functions                                                                         | :x:                | :x:                | :x:                | :white_check_mark: |           
| Availability Testing Span Filtering                                                                                   | :x:                | :x:                | :x:                | :white_check_mark: |
| Autopopulation of user ID, authenticated user ID, and user IP                                                         | :x:                | :x:                | :x:                | :white_check_mark: |
| Manually override/set operation name, user ID, or authenticated user ID                                               | :x:                | :x:                | :x:                | :white_check_mark: |
| [Adaptive sampling](../app/sampling.md#adaptive-sampling)                                                             | :x:                | :x:                | :x:                | :white_check_mark: |
| [Profiler](../profiler/profiler-overview.md)                                                                          | :x:                | :x:                | :x:                | :warning:          |
| [Snapshot Debugger](../snapshot-debugger/snapshot-debugger.md)                                                        | :x:                | :x:                | :x:                | :x:                |

**Key**
- :white_check_mark: This feature is available to all customers with formal support.
- :warning: This feature is available as a public preview. See [Supplemental terms of use for Microsoft Azure previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
- :x: This feature isn't available or isn't applicable.

### Can OpenTelemetry be used for web browsers?

Yes, but we don't recommend it and Azure doesn't support it. OpenTelemetry JavaScript is heavily optimized for Node.js. Instead, we recommend using the Application Insights JavaScript SDK.

### When can we expect the OpenTelemetry SDK to be available for use in web browsers?

The OpenTelemetry web SDK doesn't have a determined availability timeline. We're likely several years away from a browser SDK that is a viable alternative to the Application Insights JavaScript SDK.

### Can I test OpenTelemetry in a web browser today?

The [OpenTelemetry web sandbox](https://github.com/open-telemetry/opentelemetry-sandbox-web-js) is a fork designed to make OpenTelemetry work in a browser. It's not yet possible to send telemetry to Application Insights. The SDK doesn't define general client events.

### Is running Application Insights alongside competitor agents like AppDynamics, DataDog, and NewRelic supported?

No. This practice isn't something we plan to test or support, although our Distros allow you to [export to an OTLP endpoint](../app/opentelemetry-configuration.md#enable-the-otlp-exporter) alongside Azure Monitor simultaneously.

### Can I use preview features in production environments?

We don't recommend it. See [Supplemental terms of use for Microsoft Azure previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

### What's the difference between manual and automatic instrumentation?

See the [OpenTelemetry Overview](../app/opentelemetry-overview.md#instrumentation-options).

### Can I use the OpenTelemetry Collector?

Some customers use the [OpenTelemetry Collector](https://github.com/open-telemetry/opentelemetry-collector/blob/main/docs/design.md) as an agent alternative, even though Microsoft doesn't officially support an agent-based approach for application monitoring yet. In the meantime, the open-source community contributed an [OpenTelemetry Collector Azure Monitor Exporter](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/exporter/azuremonitorexporter) that some customers are using to send data to Azure Monitor Application Insights. **This is not supported by Microsoft.**

### What's the difference between OpenCensus and OpenTelemetry?

[OpenCensus](https://opencensus.io/) is the precursor to [OpenTelemetry](https://opentelemetry.io/). Microsoft helped bring together [OpenTracing](https://opentracing.io/) and OpenCensus to create OpenTelemetry, a single observability standard for the world. The current [production-recommended Python SDK](/previous-versions/azure/azure-monitor/app/opencensus-python) for Azure Monitor is based on OpenCensus. Microsoft is committed to making Azure Monitor based on OpenTelemetry.
