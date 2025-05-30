# Kafka Output Plugin

This plugin writes metrics to a [Kafka Broker][kafka] acting a Kafka Producer.

⭐ Telegraf v0.1.7
🏷️ messaging
💻 all

[kafka]: http://kafka.apache.org

## Global configuration options <!-- @/docs/includes/plugin_config.md -->

In addition to the plugin-specific configuration settings, plugins support
additional global and plugin configuration settings. These settings are used to
modify metrics, tags, and field or create aliases and configure ordering, etc.
See the [CONFIGURATION.md][CONFIGURATION.md] for more details.

[CONFIGURATION.md]: ../../../docs/CONFIGURATION.md#plugins

## Startup error behavior options <!-- @/docs/includes/startup_error_behavior.md -->

In addition to the plugin-specific and global configuration settings the plugin
supports options for specifying the behavior when experiencing startup errors
using the `startup_error_behavior` setting. Available values are:

- `error`:  Telegraf with stop and exit in case of startup errors. This is the
            default behavior.
- `ignore`: Telegraf will ignore startup errors for this plugin and disables it
            but continues processing for all other plugins.
- `retry`:  Telegraf will try to startup the plugin in every gather or write
            cycle in case of startup errors. The plugin is disabled until
            the startup succeeds.
- `probe`:  Telegraf will probe the plugin's function (if possible) and disables the plugin
            in case probing fails. If the plugin does not support probing, Telegraf will
            behave as if `ignore` was set instead.

## Secret-store support

This plugin supports secrets from secret-stores for the `sasl_username`,
`sasl_password` and `sasl_access_token` option.
See the [secret-store documentation][SECRETSTORE] for more details on how
to use them.

[SECRETSTORE]: ../../../docs/CONFIGURATION.md#secret-store-secrets

## Configuration

```toml @sample.conf
# Configuration for the Kafka server to send metrics to
[[outputs.kafka]]
  ## URLs of kafka brokers
  ## The brokers listed here are used to connect to collect metadata about a
  ## cluster. However, once the initial metadata collect is completed, telegraf
  ## will communicate solely with the kafka leader and not all defined brokers.
  brokers = ["localhost:9092"]

  ## Kafka topic for producer messages
  topic = "telegraf"

  ## The value of this tag will be used as the topic.  If not set the 'topic'
  ## option is used.
  # topic_tag = ""

  ## If true, the 'topic_tag' will be removed from to the metric.
  # exclude_topic_tag = false

  ## Optional Client id
  # client_id = "Telegraf"

  ## Set the minimal supported Kafka version.  Setting this enables the use of new
  ## Kafka features and APIs.  Of particular interested, lz4 compression
  ## requires at least version 0.10.0.0.
  ##   ex: version = "1.1.0"
  # version = ""

  ## The routing tag specifies a tagkey on the metric whose value is used as
  ## the message key.  The message key is used to determine which partition to
  ## send the message to.  This tag is preferred over the routing_key option.
  routing_tag = "host"

  ## The routing key is set as the message key and used to determine which
  ## partition to send the message to.  This value is only used when no
  ## routing_tag is set or as a fallback when the tag specified in routing tag
  ## is not found.
  ##
  ## If set to "random", a random value will be generated for each message.
  ##
  ## When unset, no message key is added and each message is routed to a random
  ## partition.
  ##
  ##   ex: routing_key = "random"
  ##       routing_key = "telegraf"
  # routing_key = ""

  ## Compression codec represents the various compression codecs recognized by
  ## Kafka in messages.
  ##  0 : None
  ##  1 : Gzip
  ##  2 : Snappy
  ##  3 : LZ4
  ##  4 : ZSTD
  # compression_codec = 0

  ## Idempotent Writes
  ## If enabled, exactly one copy of each message is written.
  # idempotent_writes = false

  ##  RequiredAcks is used in Produce Requests to tell the broker how many
  ##  replica acknowledgements it must see before responding
  ##   0 : the producer never waits for an acknowledgement from the broker.
  ##       This option provides the lowest latency but the weakest durability
  ##       guarantees (some data will be lost when a server fails).
  ##   1 : the producer gets an acknowledgement after the leader replica has
  ##       received the data. This option provides better durability as the
  ##       client waits until the server acknowledges the request as successful
  ##       (only messages that were written to the now-dead leader but not yet
  ##       replicated will be lost).
  ##   -1: the producer gets an acknowledgement after all in-sync replicas have
  ##       received the data. This option provides the best durability, we
  ##       guarantee that no messages will be lost as long as at least one in
  ##       sync replica remains.
  # required_acks = -1

  ## The maximum number of times to retry sending a metric before failing
  ## until the next flush.
  # max_retry = 3

  ## The maximum permitted size of a message. Should be set equal to or
  ## smaller than the broker's 'message.max.bytes'.
  # max_message_bytes = 1000000

  ## Producer timestamp
  ## This option sets the timestamp of the kafka producer message, choose from:
  ##   * metric: Uses the metric's timestamp
  ##   * now: Uses the time of write
  # producer_timestamp = metric

  ## Add metric name as specified kafka header if not empty
  # metric_name_header = ""

  ## Optional TLS Config
  # enable_tls = false
  # tls_ca = "/etc/telegraf/ca.pem"
  # tls_cert = "/etc/telegraf/cert.pem"
  # tls_key = "/etc/telegraf/key.pem"
  ## Use TLS but skip chain & host verification
  # insecure_skip_verify = false

  ## Period between keep alive probes.
  ## Defaults to the OS configuration if not specified or zero.
  # keep_alive_period = "15s"

  ## Optional SOCKS5 proxy to use when connecting to brokers
  # socks5_enabled = true
  # socks5_address = "127.0.0.1:1080"
  # socks5_username = "alice"
  # socks5_password = "pass123"

  ## Optional SASL Config
  # sasl_username = ""
  # sasl_password = ""

  ## Optional SASL, one of:
  ##   OAUTHBEARER, PLAIN, SCRAM-SHA-256, SCRAM-SHA-512, GSSAPI, AWS-MSK-IAM
  # sasl_mechanism = ""

  ## used if sasl_mechanism is GSSAPI
  # sasl_gssapi_service_name = ""
  # ## One of: KRB5_USER_AUTH and KRB5_KEYTAB_AUTH
  # sasl_gssapi_auth_type = "KRB5_USER_AUTH"
  # sasl_gssapi_kerberos_config_path = "/"
  # sasl_gssapi_realm = "realm"
  # sasl_gssapi_key_tab_path = ""
  # sasl_gssapi_disable_pafxfast = false

  ## Access token used if sasl_mechanism is OAUTHBEARER
  # sasl_access_token = ""

  ## used if sasl_mechanism is AWS-MSK-IAM
  # sasl_aws_msk_iam_region = ""
  ## for profile based auth
  ## sasl_aws_msk_iam_profile = ""
  ## for role based auth
  ## sasl_aws_msk_iam_role = ""
  ## sasl_aws_msk_iam_session = ""

  ## Arbitrary key value string pairs to pass as a TOML table. For example:
  ## {logicalCluster = "cluster-042", poolId = "pool-027"}
  # sasl_extensions = {}

  ## SASL protocol version.  When connecting to Azure EventHub set to 0.
  # sasl_version = 1

  # Disable Kafka metadata full fetch
  # metadata_full = false

  ## Maximum number of retries for metadata operations including
  ## connecting. Sets Sarama library's Metadata.Retry.Max config value. If 0 or
  ## unset, use the Sarama default of 3,
  # metadata_retry_max = 0

  ## Type of retry backoff. Valid options: "constant", "exponential"
  # metadata_retry_type = "constant"

  ## Amount of time to wait before retrying. When metadata_retry_type is
  ## "constant", each retry is delayed this amount. When "exponential", the
  ## first retry is delayed this amount, and subsequent delays are doubled. If 0
  ## or unset, use the Sarama default of 250 ms
  # metadata_retry_backoff = 0

  ## Maximum amount of time to wait before retrying when metadata_retry_type is
  ## "exponential". Ignored for other retry types. If 0, there is no backoff
  ## limit.
  # metadata_retry_max_duration = 0

  ## Data format to output.
  ## Each data format has its own unique set of configuration options, read
  ## more about them here:
  ## https://github.com/influxdata/telegraf/blob/master/docs/DATA_FORMATS_OUTPUT.md
  # data_format = "influx"

  ## NOTE: Due to the way TOML is parsed, tables must be at the END of the
  ## plugin definition, otherwise additional config options are read as part of
  ## the table

  ## Optional topic suffix configuration.
  ## If the section is omitted, no suffix is used.
  ## Following topic suffix methods are supported:
  ##   measurement - suffix equals to separator + measurement's name
  ##   tags        - suffix equals to separator + specified tags' values
  ##                 interleaved with separator

  ## Suffix equals to "_" + measurement name
  # [outputs.kafka.topic_suffix]
  #   method = "measurement"
  #   separator = "_"

  ## Suffix equals to "__" + measurement's "foo" tag value.
  ## If there's no such a tag, suffix equals to an empty string
  # [outputs.kafka.topic_suffix]
  #   method = "tags"
  #   keys = ["foo"]
  #   separator = "__"

  ## Suffix equals to "_" + measurement's "foo" and "bar"
  ## tag values, separated by "_". If there is no such tags,
  ## their values treated as empty strings.
  # [outputs.kafka.topic_suffix]
  #   method = "tags"
  #   keys = ["foo", "bar"]
  #   separator = "_"
```

### `max_retry`

This option controls the number of retries before a failure notification is
displayed for each message when no acknowledgement is received from the
broker. When the setting is greater than `0`, message latency can be reduced,
duplicate messages can occur in cases of transient errors, and broker loads can
increase during downtime.

The option is similar to the
[retries](https://kafka.apache.org/documentation/#producerconfigs) Producer
option in the Java Kafka Producer.
