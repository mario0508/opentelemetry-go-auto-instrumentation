# Environment Variables Configuration Guide

This guide provides comprehensive information about configuring OpenTelemetry Go Auto-Instrumentation using environment variables.

## Overview

The OpenTelemetry Go Auto-Instrumentation tool supports configuration through environment variables, which can override default settings and provide runtime configuration without modifying command-line arguments.

## Tool-Specific Environment Variables

The `otel` tool itself supports the following environment variables with the `OTELTOOL_` prefix:

| Environment Variable | Description | Type | Example |
|---------------------|-------------|------|---------|
| `OTELTOOL_DEBUG` | Enable debug mode | boolean | `true` or `false` |
| `OTELTOOL_VERBOSE` | Enable verbose logging | boolean | `true` or `false` |
| `OTELTOOL_RULE_JSON_FILES` | Specify custom rule files | string | `rule1.json,rule2.json` |
| `OTELTOOL_DISABLE_DEFAULT` | Disable default rules | boolean | `true` or `false` |
| `OTELTOOL_LOG` | Custom log file path | string | `/path/to/file.log` |

## OpenTelemetry SDK Environment Variables

The instrumented application will also respect standard OpenTelemetry environment variables:

### Service Configuration
- `OTEL_SERVICE_NAME`: Your service name
- `OTEL_SERVICE_VERSION`: Your service version
- `OTEL_SERVICE_NAMESPACE`: Your service namespace

### Exporter Configuration
- `OTEL_EXPORTER_OTLP_ENDPOINT`: OTLP endpoint for traces and metrics
- `OTEL_EXPORTER_OTLP_TRACES_ENDPOINT`: OTLP endpoint for traces only
- `OTEL_EXPORTER_OTLP_METRICS_ENDPOINT`: OTLP endpoint for metrics only
- `OTEL_EXPORTER_OTLP_HEADERS`: Headers for OTLP requests
- `OTEL_EXPORTER_OTLP_PROTOCOL`: Protocol for OTLP (`grpc` or `http/protobuf`)

### Timeout and Duration Configuration

⚠️ **Important**: All timeout and duration values must include time unit suffixes.

#### Supported Time Units
- `ns` - nanoseconds
- `us` or `µs` - microseconds  
- `ms` - milliseconds
- `s` - seconds
- `m` - minutes
- `h` - hours

#### Common Duration Environment Variables
| Variable | Description | Valid Examples | Invalid Examples |
|----------|-------------|----------------|------------------|
| `OTEL_EXPORTER_OTLP_TIMEOUT` | OTLP exporter timeout | `30s`, `5000ms`, `1m` | `30`, `5000` |
| `OTEL_EXPORTER_JAEGER_TIMEOUT` | Jaeger exporter timeout | `10s`, `2000ms` | `10`, `2000` |
| `OTEL_METRIC_EXPORT_TIMEOUT` | Metric export timeout | `15s`, `15000ms` | `15`, `15000` |
| `OTEL_BSP_EXPORT_TIMEOUT` | Batch span processor export timeout | `30s`, `30000ms` | `30`, `30000` |

## Common Configuration Examples

### Basic Configuration
```bash
export OTEL_SERVICE_NAME="my-service"
export OTEL_EXPORTER_OTLP_ENDPOINT="http://localhost:4317"
export OTEL_EXPORTER_OTLP_TIMEOUT="30s"
```

### Debug Configuration
```bash
export OTELTOOL_DEBUG=true
export OTELTOOL_VERBOSE=true
export OTELTOOL_LOG="/tmp/otel-debug.log"
```

### Custom Rules Configuration
```bash
export OTELTOOL_DISABLE_DEFAULT=true
export OTELTOOL_RULE_JSON_FILES="custom-rules.json,additional-rules.json"
```

### Production Configuration
```bash
export OTEL_SERVICE_NAME="production-api"
export OTEL_SERVICE_VERSION="1.2.3"
export OTEL_EXPORTER_OTLP_ENDPOINT="https://otlp.example.com:4317"
export OTEL_EXPORTER_OTLP_TIMEOUT="10s"
export OTEL_EXPORTER_OTLP_PROTOCOL="grpc"
export OTEL_EXPORTER_OTLP_HEADERS="authorization=Bearer token123"
```

## Troubleshooting

### Duration Parsing Errors

If you encounter errors like:
```
env: parse error on field "Timeout" of type "time.Duration": unable to parse duration: time: missing unit in duration "4000"
```

**Cause**: You've provided a numeric value without a time unit suffix.

**Solution**: Add the appropriate time unit suffix:
- Change `4000` to `4000ms` (4 seconds in milliseconds)
- Or `4s` (4 seconds)
- Or `4000000000ns` (4 seconds in nanoseconds)

**Examples of fixes**:
```bash
# ❌ Incorrect
export OTEL_EXPORTER_OTLP_TIMEOUT=4000

# ✅ Correct
export OTEL_EXPORTER_OTLP_TIMEOUT=4000ms
# or
export OTEL_EXPORTER_OTLP_TIMEOUT=4s
```

### Environment Variable Priority

Environment variables take precedence over configuration files but may be overridden by command-line flags:

1. Command-line flags (highest priority)
2. Environment variables
3. Configuration files (lowest priority)

### Validation

To verify your environment variables are correctly set:

```bash
# Check tool-specific variables
env | grep OTELTOOL_

# Check OpenTelemetry variables  
env | grep OTEL_

# Test duration parsing
echo $OTEL_EXPORTER_OTLP_TIMEOUT
```

### Debugging Environment Issues

1. **Enable verbose logging**:
   ```bash
   export OTELTOOL_VERBOSE=true
   export OTELTOOL_DEBUG=true
   ```

2. **Check logs** for configuration parsing messages

3. **Verify variable names** - they are case-sensitive

4. **Test with minimal configuration** first, then add complexity

## Best Practices

1. **Always include time units** for duration values
2. **Use consistent naming** for service identification  
3. **Set appropriate timeouts** based on your network conditions
4. **Use environment-specific configurations** (dev, staging, prod)
5. **Document your environment variables** in your project README
6. **Validate configurations** in your deployment pipeline
7. **Use secrets management** for sensitive values like API keys

## Integration with Deployment Tools

### Docker
```dockerfile
ENV OTEL_SERVICE_NAME=my-service
ENV OTEL_EXPORTER_OTLP_TIMEOUT=30s
ENV OTEL_EXPORTER_OTLP_ENDPOINT=http://jaeger:4317
```

### Kubernetes
```yaml
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    spec:
      containers:
      - name: app
        env:
        - name: OTEL_SERVICE_NAME
          value: "my-service"
        - name: OTEL_EXPORTER_OTLP_TIMEOUT
          value: "30s"
        - name: OTEL_EXPORTER_OTLP_ENDPOINT
          value: "http://jaeger:4317"
```

### Docker Compose
```yaml
version: '3.8'
services:
  app:
    environment:
      - OTEL_SERVICE_NAME=my-service
      - OTEL_EXPORTER_OTLP_TIMEOUT=30s
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://jaeger:4317
```
