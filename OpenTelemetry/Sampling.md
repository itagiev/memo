## Sampling

Позволяет урезать набор данных, чтобы уменьшить потребление памяти.
Желательно не использовать, если нет необходимости.

### Head sampling

1. Application level
2. Remove application overhead

```C#
// Пример

public class RateSampler : Sampler
{
    private readonly double _samplingRate;

    public RateSampler(double samplingRate) => _samplingRate = samplingRate;

    public override SamplingResult ShouldSample(in SamplingParameters samplingParameters)
    {
        var shouldBeSampled = Random.Shared.NextDouble() < _samplingRate;
        if (shouldBeSampled)
            return new SamplingResult(SamplingDecision.RecordAndSample);
        return new SamplingResult(SamplingDecision.Drop);
    }
}

builder.Services.AddOpenTelemetry()
    .WithTracing(tracing =>
    tracing
        .SetSampler(new RateSampler(0.25d)));

// Или

builder.Services.ConfigureOpenTelemetryTracerProvider(options =>
{
    options.SetSampler(new RateSampler(0.25));
});
```

### Tail-based sampling

1. Collector level
2. After all traces from a request come in
3. Informed sampling decisions

```yml
tail_sampling:
  decision_wait: 10s
  expected_new_traces_per_second: 10
  policies:
    - name: string_attribute_policy
      type: string_attribute
      string_attribute:
        key: http.route
        values:
        - ^\/healthz$
        enabled_regex_matching: true
        invert_match: true
```

### Storage-based sampling

???