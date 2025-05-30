# OpenTelemetry Input Plugin

This service plugin receives traces, metrics, logs and profiles from
[OpenTelemetry][opentelemetry] clients and compatible agents via gRPC.

⭐ Telegraf v1.19.0
🏷️ logging, messaging
💻 all

[opentelemetry]: https://opentelemetry.io

## Service Input <!-- @/docs/includes/service_input.md -->

This plugin is a service input. Normal plugins gather metrics determined by the
interval setting. Service plugins start a service to listen and wait for
metrics or events to occur. Service plugins have two key differences from
normal plugins:

1. The global or plugin specific `interval` setting may not apply
2. The CLI options of `--test`, `--test-wait`, and `--once` may not produce
   output for this plugin

## Global configuration options <!-- @/docs/includes/plugin_config.md -->

In addition to the plugin-specific configuration settings, plugins support
additional global and plugin configuration settings. These settings are used to
modify metrics, tags, and field or create aliases and configure ordering, etc.
See the [CONFIGURATION.md][CONFIGURATION.md] for more details.

[CONFIGURATION.md]: ../../../docs/CONFIGURATION.md#plugins

## Configuration

```toml @sample.conf
# Receive OpenTelemetry traces, metrics, and logs over gRPC
[[inputs.opentelemetry]]
  ## Override the default (0.0.0.0:4317) destination OpenTelemetry gRPC service
  ## address:port
  # service_address = "0.0.0.0:4317"

  ## Override the default (5s) new connection timeout
  # timeout = "5s"

  ## gRPC Maximum Message Size
  # max_msg_size = "4MB"

  ## Override the default span attributes to be used as line protocol tags.
  ## These are always included as tags:
  ## - trace ID
  ## - span ID
  ## Common attributes can be found here:
  ## - https://github.com/open-telemetry/opentelemetry-collector/tree/main/semconv
  # span_dimensions = ["service.name", "span.name"]

  ## Override the default log record attributes to be used as line protocol tags.
  ## These are always included as tags, if available:
  ## - trace ID
  ## - span ID
  ## Common attributes can be found here:
  ## - https://github.com/open-telemetry/opentelemetry-collector/tree/main/semconv
  ## When using InfluxDB for both logs and traces, be certain that log_record_dimensions
  ## matches the span_dimensions value.
  # log_record_dimensions = ["service.name"]

  ## Override the default profile attributes to be used as line protocol tags.
  ## These are always included as tags, if available:
  ## - profile_id
  ## - address
  ## - sample
  ## - sample_name
  ## - sample_unit
  ## - sample_type
  ## - sample_type_unit
  ## Common attributes can be found here:
  ## - https://github.com/open-telemetry/opentelemetry-collector/tree/main/semconv
  # profile_dimensions = []

  ## Override the default (prometheus-v1) metrics schema.
  ## Supports: "prometheus-v1", "prometheus-v2"
  ## For more information about the alternatives, read the Prometheus input
  ## plugin notes.
  # metrics_schema = "prometheus-v1"

  ## Optional TLS Config.
  ## For advanced options: https://github.com/influxdata/telegraf/blob/v1.18.3/docs/TLS.md
  ##
  ## Set one or more allowed client CA certificate file names to
  ## enable mutually authenticated TLS connections.
  # tls_allowed_cacerts = ["/etc/telegraf/clientca.pem"]
  ## Add service certificate and key.
  # tls_cert = "/etc/telegraf/cert.pem"
  # tls_key = "/etc/telegraf/key.pem"
```

### Schema

The OpenTelemetry->InfluxDB conversion [schema][1] and [implementation][2] are
hosted at <https://github.com/influxdata/influxdb-observability> .

Spans are stored in measurement `spans`.
Logs are stored in measurement `logs`.

For metrics, two output schemata exist.  Metrics received with
`metrics_schema=prometheus-v1` are assigned measurement from the OTel field
`Metric.name`.  Metrics received with `metrics_schema=prometheus-v2` are stored
in measurement `prometheus`.

Also see the OpenTelemetry output plugin for Telegraf.

[1]: https://github.com/influxdata/influxdb-observability/blob/main/docs/index.md

[2]: https://github.com/influxdata/influxdb-observability/tree/main/otel2influx

## Example Output

### Tracing Spans

```text
spans end_time_unix_nano="2021-02-19 20:50:25.6893952 +0000 UTC",instrumentation_library_name="tracegen",kind="SPAN_KIND_INTERNAL",name="okey-dokey",net.peer.ip="1.2.3.4",parent_span_id="d5270e78d85f570f",peer.service="tracegen-client",service.name="tracegen",span.kind="server",span_id="4c28227be6a010e1",status_code="STATUS_CODE_OK",trace_id="7d4854815225332c9834e6dbf85b9380" 1613767825689169000
spans end_time_unix_nano="2021-02-19 20:50:25.6893952 +0000 UTC",instrumentation_library_name="tracegen",kind="SPAN_KIND_INTERNAL",name="lets-go",net.peer.ip="1.2.3.4",peer.service="tracegen-server",service.name="tracegen",span.kind="client",span_id="d5270e78d85f570f",status_code="STATUS_CODE_OK",trace_id="7d4854815225332c9834e6dbf85b9380" 1613767825689135000
spans end_time_unix_nano="2021-02-19 20:50:25.6895667 +0000 UTC",instrumentation_library_name="tracegen",kind="SPAN_KIND_INTERNAL",name="okey-dokey",net.peer.ip="1.2.3.4",parent_span_id="b57e98af78c3399b",peer.service="tracegen-client",service.name="tracegen",span.kind="server",span_id="a0643a156d7f9f7f",status_code="STATUS_CODE_OK",trace_id="fd6b8bb5965e726c94978c644962cdc8" 1613767825689388000
spans end_time_unix_nano="2021-02-19 20:50:25.6895667 +0000 UTC",instrumentation_library_name="tracegen",kind="SPAN_KIND_INTERNAL",name="lets-go",net.peer.ip="1.2.3.4",peer.service="tracegen-server",service.name="tracegen",span.kind="client",span_id="b57e98af78c3399b",status_code="STATUS_CODE_OK",trace_id="fd6b8bb5965e726c94978c644962cdc8" 1613767825689303300
spans end_time_unix_nano="2021-02-19 20:50:25.6896741 +0000 UTC",instrumentation_library_name="tracegen",kind="SPAN_KIND_INTERNAL",name="okey-dokey",net.peer.ip="1.2.3.4",parent_span_id="6a8e6a0edcc1c966",peer.service="tracegen-client",service.name="tracegen",span.kind="server",span_id="d68f7f3b41eb8075",status_code="STATUS_CODE_OK",trace_id="651dadde186b7834c52b13a28fc27bea" 1613767825689480300
```

## Metrics

### `prometheus-v1`

```text
cpu_temp,foo=bar gauge=87.332
http_requests_total,method=post,code=200 counter=1027
http_requests_total,method=post,code=400 counter=3
http_request_duration_seconds 0.05=24054,0.1=33444,0.2=100392,0.5=129389,1=133988,sum=53423,count=144320
rpc_duration_seconds 0.01=3102,0.05=3272,0.5=4773,0.9=9001,0.99=76656,sum=1.7560473e+07,count=2693
```

### `prometheus-v2`

```text
prometheus,foo=bar cpu_temp=87.332
prometheus,method=post,code=200 http_requests_total=1027
prometheus,method=post,code=400 http_requests_total=3
prometheus,le=0.05 http_request_duration_seconds_bucket=24054
prometheus,le=0.1  http_request_duration_seconds_bucket=33444
prometheus,le=0.2  http_request_duration_seconds_bucket=100392
prometheus,le=0.5  http_request_duration_seconds_bucket=129389
prometheus,le=1    http_request_duration_seconds_bucket=133988
prometheus         http_request_duration_seconds_count=144320,http_request_duration_seconds_sum=53423
prometheus,quantile=0.01 rpc_duration_seconds=3102
prometheus,quantile=0.05 rpc_duration_seconds=3272
prometheus,quantile=0.5  rpc_duration_seconds=4773
prometheus,quantile=0.9  rpc_duration_seconds=9001
prometheus,quantile=0.99 rpc_duration_seconds=76656
prometheus               rpc_duration_seconds_count=1.7560473e+07,rpc_duration_seconds_sum=2693
```

### Logs

```text
logs fluent.tag="fluent.info",pid=18i,ppid=9i,worker=0i 1613769568895331700
logs fluent.tag="fluent.debug",instance=1720i,queue_size=0i,stage_size=0i 1613769568895697200
logs fluent.tag="fluent.info",worker=0i 1613769568896515100
```

### Profiles

```text
profiles,address=95210353,host.name=testbox,profile_id=618098d29a6cefd6a4c0ea806880c2a8,sample=0,sample_name=cpu,sample_type=samples,sample_type_unit=count,sample_unit=nanoseconds build_id="fab9b8c848218405738c11a7ec4982e9",build_id_type="BUILD_ID_BINARY_HASH",end_time_unix_nano=1721306050081621681u,file_offset=18694144u,filename="chromium",frame_type="native",location="",memory_limit=250413056u,memory_start=18698240u,stack_trace_id="hYmAzQVF8vy8MWbzsKpQNw",start_time_unix_nano=1721306050081621681u,value=1i 1721306048731622020
profiles,address=15945263,host.name=testbox,profile_id=618098d29a6cefd6a4c0ea806880c2a8,sample=1,sample_name=cpu,sample_type=samples,sample_type_unit=count,sample_unit=nanoseconds build_id="7dab4a2e0005d025e75cc72191f8d6bf",build_id_type="BUILD_ID_BINARY_HASH",end_time_unix_nano=1721306050081621681u,file_offset=15638528u,filename="dockerd",frame_type="native",location="",memory_limit=47255552u,memory_start=15638528u,stack_trace_id="4N3KEcGylb5Qoi2905c1ZA",start_time_unix_nano=1721306050081621681u,value=1i 1721306049831718725
profiles,address=15952400,host.name=testbox,profile_id=618098d29a6cefd6a4c0ea806880c2a8,sample=1,sample_name=cpu,sample_type=samples,sample_type_unit=count,sample_unit=nanoseconds build_id="7dab4a2e0005d025e75cc72191f8d6bf",build_id_type="BUILD_ID_BINARY_HASH",end_time_unix_nano=1721306050081621681u,file_offset=15638528u,filename="dockerd",frame_type="native",location="",memory_limit=47255552u,memory_start=15638528u,stack_trace_id="4N3KEcGylb5Qoi2905c1ZA",start_time_unix_nano=1721306050081621681u,value=1i 1721306049831718725
profiles,address=15953899,host.name=testbox,profile_id=618098d29a6cefd6a4c0ea806880c2a8,sample=1,sample_name=cpu,sample_type=samples,sample_type_unit=count,sample_unit=nanoseconds build_id="7dab4a2e0005d025e75cc72191f8d6bf",build_id_type="BUILD_ID_BINARY_HASH",end_time_unix_nano=1721306050081621681u,file_offset=15638528u,filename="dockerd",frame_type="native",location="",memory_limit=47255552u,memory_start=15638528u,stack_trace_id="4N3KEcGylb5Qoi2905c1ZA",start_time_unix_nano=1721306050081621681u,value=1i 1721306049831718725
profiles,address=16148175,host.name=testbox,profile_id=618098d29a6cefd6a4c0ea806880c2a8,sample=1,sample_name=cpu,sample_type=samples,sample_type_unit=count,sample_unit=nanoseconds build_id="7dab4a2e0005d025e75cc72191f8d6bf",build_id_type="BUILD_ID_BINARY_HASH",end_time_unix_nano=1721306050081621681u,file_offset=15638528u,filename="dockerd",frame_type="native",location="",memory_limit=47255552u,memory_start=15638528u,stack_trace_id="4N3KEcGylb5Qoi2905c1ZA",start_time_unix_nano=1721306050081621681u,value=1i 1721306049831718725
profiles,address=4770577,host.name=testbox,profile_id=618098d29a6cefd6a4c0ea806880c2a8,sample=2,sample_name=cpu,sample_type=samples,sample_type_unit=count,sample_unit=nanoseconds build_id="cfc3dc7d1638c1284a6b62d4b5c0d74e",build_id_type="BUILD_ID_BINARY_HASH",end_time_unix_nano=1721306050081621681u,file_offset=0u,filename="",frame_type="kernel",location="do_epoll_wait",memory_limit=0u,memory_start=0u,stack_trace_id="UaO9bysJnAYXFYobSdHXqg",start_time_unix_nano=1721306050081621681u,value=1i 1721306050081621681
profiles,address=4773632,host.name=testbox,profile_id=618098d29a6cefd6a4c0ea806880c2a8,sample=2,sample_name=cpu,sample_type=samples,sample_type_unit=count,sample_unit=nanoseconds build_id="cfc3dc7d1638c1284a6b62d4b5c0d74e",build_id_type="BUILD_ID_BINARY_HASH",end_time_unix_nano=1721306050081621681u,file_offset=0u,filename="",frame_type="kernel",location="__x64_sys_epoll_wait",memory_limit=0u,memory_start=0u,stack_trace_id="UaO9bysJnAYXFYobSdHXqg",start_time_unix_nano=1721306050081621681u,value=1i 1721306050081621681
profiles,address=14783666,host.name=testbox,profile_id=618098d29a6cefd6a4c0ea806880c2a8,sample=2,sample_name=cpu,sample_type=samples,sample_type_unit=count,sample_unit=nanoseconds build_id="cfc3dc7d1638c1284a6b62d4b5c0d74e",build_id_type="BUILD_ID_BINARY_HASH",end_time_unix_nano=1721306050081621681u,file_offset=0u,filename="",frame_type="kernel",location="do_syscall_64",memory_limit=0u,memory_start=0u,stack_trace_id="UaO9bysJnAYXFYobSdHXqg",start_time_unix_nano=1721306050081621681u,value=1i 1721306050081621681
profiles,address=16777518,host.name=testbox,profile_id=618098d29a6cefd6a4c0ea806880c2a8,sample=2,sample_name=cpu,sample_type=samples,sample_type_unit=count,sample_unit=nanoseconds build_id="cfc3dc7d1638c1284a6b62d4b5c0d74e",build_id_type="BUILD_ID_BINARY_HASH",end_time_unix_nano=1721306050081621681u,file_offset=0u,filename="",frame_type="kernel",location="entry_SYSCALL_64_after_hwframe",memory_limit=0u,memory_start=0u,stack_trace_id="UaO9bysJnAYXFYobSdHXqg",start_time_unix_nano=1721306050081621681u,value=1i 1721306050081621681
profiles,address=1139937,host.name=testbox,profile_id=618098d29a6cefd6a4c0ea806880c2a8,sample=2,sample_name=cpu,sample_type=samples,sample_type_unit=count,sample_unit=nanoseconds build_id="982ed6c7a77f99f0ae746be0187953bf",build_id_type="BUILD_ID_BINARY_HASH",end_time_unix_nano=1721306050081621681u,file_offset=147456u,filename="libc.so.6",frame_type="native",location="",memory_limit=1638400u,memory_start=147456u,stack_trace_id="UaO9bysJnAYXFYobSdHXqg",start_time_unix_nano=1721306050081621681u,value=1i 1721306050081621681
profiles,address=117834912,host.name=testbox,profile_id=618098d29a6cefd6a4c0ea806880c2a8,sample=2,sample_name=cpu,sample_type=samples,sample_type_unit=count,sample_unit=nanoseconds build_id="fab9b8c848218405738c11a7ec4982e9",build_id_type="BUILD_ID_BINARY_HASH",end_time_unix_nano=1721306050081621681u,file_offset=18694144u,filename="chromium",frame_type="native",location="",memory_limit=250413056u,memory_start=18698240u,stack_trace_id="UaO9bysJnAYXFYobSdHXqg",start_time_unix_nano=1721306050081621681u,value=1i 1721306050081621681
```
