# Span Processor

| Status                   |                                                 |
| ------------------------ |-------------------------------------------------|
| Stability                | [alpha]                                         |
| Supported pipeline types | traces                                          |
| Distributions            | [core], [contrib], [observiq], [splunk], [sumo] |

The span processor modifies the span name based on its attributes or extract span attributes from the span name. It also allows
to change span status. Please refer to [config.go](./config.go) for the config spec.

It optionally supports the ability to [include/exclude spans](../attributesprocessor/README.md#includeexclude-filtering).

The following actions are supported:

- `name`: Modify the name of attributes within a span
- `status`: Modify the status of the span

### Name a span

The following settings are required:

- `from_attributes`: The attribute value for the keys are used to create a
new name in the order specified in the configuration.

The following settings can be optionally configured:

- `separator`: A string, which is specified will be used to split values

Note: If renaming is dependent on attributes being modified by the `attributes`
processor, ensure the `span` processor is specified after the `attributes`
processor in the `pipeline` specification.

```yaml
span:
  name:
    # from_attributes represents the attribute keys to pull the values from to generate the
    # new span name.
    from_attributes: [<key1>, <key2>, ...]
    # Separator is the string used to concatenate various parts of the span name.
    separator: <value>
```

Example:

```yaml
span:
  name:
    from_attributes: ["db.svc", "operation"]
    separator: "::"
```

Refer to [config.yaml](./testdata/config.yaml) for detailed
examples on using the processor.

### Extract attributes from span name

Takes a list of regular expressions to match span name against and extract
attributes from it based on subexpressions. Must be specified under the
`to_attributes` section.

The following settings are required:

- `rules`: A list of rules to extract attribute values from span name. The values
in the span name are replaced by extracted attribute names. Each rule in the list
is regex pattern string. Span name is checked against the regex and if the regex
matches then all named subexpressions of the regex are extracted as attributes
and are added to the span. Each subexpression name becomes an attribute name and
subexpression matched portion becomes the attribute value. The matched portion
in the span name is replaced by extracted attribute name. If the attributes
already exist in the span then they will be overwritten. The process is repeated
for all rules in the order they are specified. Each subsequent rule works on the
span name that is the output after processing the previous rule.
- `break_after_match` (default = false): specifies if processing of rules should stop after the first
match. If it is false rule processing will continue to be performed over the
modified span name.

```yaml
span/to_attributes:
  name:
    to_attributes:
      rules:
        - regexp-rule1
        - regexp-rule2
        - regexp-rule3
        ...
      break_after_match: <true|false>

```

Example:

```yaml
# Let's assume input span name is /api/v1/document/12345678/update
# Applying the following results in output span name /api/v1/document/{documentId}/update
# and will add a new attribute "documentId"="12345678" to the span.
span/to_attributes:
  name:
    to_attributes:
      rules:
        - ^\/api\/v1\/document\/(?P<documentId>.*)\/update$
```

### Set status for span

The following setting is required:

- `code`: Represents span status. One of the following values "Unset", "Error", "Ok".

The following setting is allowed only for code "Error":
- `description`

Example:

```yaml
# Set status allows to set specific status for a given span. Possible values are
# Ok, Error and Unset as per
# https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/trace/api.md#set-status
# The description field allows to set a human-readable message for errors.
span/set_status:
  status:
    code: Error
    description: "some error description"
```


Refer to [config.yaml](./testdata/config.yaml) for detailed
examples on using the processor.

[alpha]:https://github.com/open-telemetry/opentelemetry-collector#alpha
[contrib]:https://github.com/open-telemetry/opentelemetry-collector-releases/tree/main/distributions/otelcol-contrib
[core]:https://github.com/open-telemetry/opentelemetry-collector-releases/tree/main/distributions/otelcol
[splunk]: https://github.com/signalfx/splunk-otel-collector
[observiq]: https://github.com/observIQ/observiq-otel-collector
[sumo]: https://github.com/SumoLogic/sumologic-otel-collector
