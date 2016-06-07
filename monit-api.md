
# Sample code

// A collector associates metrics to a monitored resource
Collector localhostCollector = (new Collector("localhost"))
    .withTarget(new SystemResource("localhost"))
    .withMetric(new CpuPercentMetric("CpuPercent"))
    .withMetric(new MemoryUsageMetric("MemoryUsage"));

// A sensor gathers information from collector(s),
// then aggregates and/or publishes them.
Sensor sensor = (new Sensor("SystemSensor"))
    .withPeriod(30);
    .withCollector(localhostCollector)
    .withAggregator(
      (new AverageAggregator())
        .withOutput("AverageCpuPercent")
        .withInput("CpuPercent"))
    .withPublisher(
      (new FileLoggerPublisher(outputFile))
        .withInput("AverageCpuPercent"));

# How to implement a driver

Implement a Resource and a set of Metric. Also optionnally implement aggregator(s) and publisher(s), if built-in ones are not sufficient.

The sample code above uses a "SystemResource", and metrics called "CpuPercentMetric" and "MemoryUsageMetric".

# Main API classes and interfaces

## TinomObject

public class TinomObject {
  public String name;

  public TinomObject(String name) {
    this.name = name;
  }

  public String getId() {
    return "urn:uuid:" + name;
  }
}

## Metric

With output channel(s) and control attributes (how data are gathered).
A metric extends Metric, and provides output channels.

public class Metric extends TinomObject{
  private Resource resource;

  public Metric(String name) { super(name); }

  public void setResource(Resource resource) {
    this.resource = resource;
  }

  public Resource getResource() {
    return resource;
  }

  public abstract String get(String channelName);

}

Example of specific metric implementation:

public class CpuPercentMetric extends Metric {

  // At least provide a constructor with name
  // The metric id will be set to "urn:uuid:<name>"
  public CpuPercentMetric(String name) {
    super(name);
  }

  // A call to get("CpuPercent") means the metric
  // provides an output channel called "CpuPercent"
  public String get(String channelName) {
    return getResource().get(channelName);
  }
}

## Aggregator

Extends metric, adding input channel(s).

public class Aggregator extends Metric {
  public Aggregator(String name) { super(name); }

  public Aggregator withInput(Metric metric, String channelName) {
    // Add specified channel as input
    return this;
  }
}

public class AverageAggregator extends Aggregator {
  public AverageAggregator(String name) { super(name); }

  public String get(String channelName) {
    // Compute average then return result
    return Double.toString(avg);
  }
}

## Collector

With period (polls resource every period).
A collector is used to wrap a resource to monitor. It extends Collector, which can be associated to a set of metrics.

public class Collector extends TinomObject {

  private Resource resource;
  private List<Metric> metrics;

  public Collector(String name) {
    super(name);
  }

  public Collector withTarget(Resource resource) {
    this.resource = resource;
  }

  public Collector withMetric(Metric metric) {
    metric.setResource(this.resource);
    metrics.add(metric);
  }
}

## Sensor

With period (every period, aggregates and publishes).

public class Sensor extends TinomObject {

  private List<Collector> collectors;
  private List<Aggregator> aggregators;
  private List<Publisher> publishers;

  public Sensor(String name) {
    super(name);
  }

  public Sensor withCollector(Collector collector) {
    this.collectors.add(collector);
  }

  public Sensor withAggregator(Aggregator aggregator) {
    this.aggregators.add(aggregator);
  }

  public Sensor withPublisher(Publisher publisher) {
    this.publishers.add(publisher);
  }

}

