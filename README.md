hawkular-client-python
=========================

This repository includes the necessary Python client libraries to access Hawkular remotely. Currently we only have a driver for the metrics component as it's the most mature of the components.

## Introduction

Python client to access Hawkular-Metrics, an abstraction to invoke REST-methods on the server endpoint using urllib2. No external dependencies, works with Python 2.7.x and Python 3.4.x (tested with the Python 3.4.2, might work with newer versions also)

## License and copyright

```
   Copyright 2015-2016 Red Hat, Inc. and/or its affiliates
   and other contributors.

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.
```

## Installation

To install, run ``python setup.py install``

## Usage

To use hawkular-client-python in your own program, after installation import from hawkular the class HawkularMetricsClient and instantiate it. It accepts parameters tenant_id, hostname and port. After this, push dicts with keys id, timestamp and value with put or use assistant method create to send events.

Timestamps should be in the milliseconds after epoch and numeric values should be float. The client provides a method to request current time in milliseconds, ``time_millis()``

See metrics_test.py for more detailed examples. The tests target a running docker instance of Hawkular Kettle by default (change setUp() to override).

### General

When a method wants a metric_type one can use the shortcuts of MetricType.Gauge or MetricType.Availability. For availability values, one can use Availability.Up and Availability.Down to simplify usage.

```python
>>> from hawkular.metrics import *
>>> client = HawkularMetricsClient(tenant_id='doc', port=8081)
```

### Creating and modifying metric definitions

While creating a metric definition is not required to use them, it is possible to define a custom data retention times as well as tags for each metric. To create a metric, use method create_metric_definition(metric_id, metric_type, **tags). There are assistant methods create_numeric_definition(metric_id, **tags) and create_availability_definition(metric_id, **tags) which bypass the need for MetricType definition. The only reserved keyword for tags is dataRetention, which will change the dataRetention time, other tags are used for user's metadata.

Example:

```python
>>> client.create_numeric_definition('example.metric.1', dataRetention=90, units='bytes', hostname='localhost')
True
>>> client.query_definitions(metrics.MetricType.Gauge)
[{u'tags': {u'units': u'bytes', u'hostname': u'localhost'}, u'id': u'example.metric.1', u'dataRetention': 90, u'tenantId': u'doc'}]
```

### Modifying metric definition tags

One powerful feature of Hawkular-Metrics is the tagging feature that allows one to define descriptive metadata for any metric. Tags can be added when creating a metric definition (see above), but also modified later.

Example:

```python
>>> client.create_numeric_definition('example.metric.1', dataRetention=90, units='bytes', hostname='localhost')
>>> client.query_metric_tags(MetricType.Gauge, 'example.metric.1')
{u'units': u'bytes', u'hostname': u'localhost'}
>>> client.delete_metric_tags(MetricType.Gauge, 'example.metric.1', units='bytes')
>>> client.query_metric_tags(MetricType.Gauge, 'example.metric.1')
{u'hostname': u'localhost'}
>>> client.update_metric_tags(MetricType.Gauge, 'example.metric.1', hostname='machine1', env='test')
>>> client.query_metric_tags(MetricType.Gauge, 'example.metric.1')
{u'hostname': u'machine1', u'env': u'test'}
```

### Pushing new values

All the methods that allow pushing values can accept both availability status as well as numeric values. It is possible to push multiple metrics with multiple values per metric in one call to the Hawkular-Metrics. However for convenience, methods which will push just one value for one metric is also provided. To push availability values, use MetricType.Availability and values Availability.Up and Availability.Down, otherwise the syntax is equal.

Example pushing a single value:

```python
datapoint = create_datapoint(float(4.35), time_millis())
metric = create_metric(MetricType.Gauge, 'example.metric.1', datapoint)
client.put(metric)
```

And a shortcut method for the above:

```python
client.push(MetricType.Gauge, 'example.metric.1', float(4.35))
```

Example pushing multiple values for the same metric (with given timestamp and without):

```python
>>> v1 = create_datapoint(float(2.345)) # Timestamp is autogenerated
>>> v2 = create_datapoint(float(3.45), 1429711362289) # Timestamp is given
>>> m = create_metric(MetricType.Gauge, 'example.metric.2', [v1, v2])
>>> client.put(m)
>>> client.query_single_numeric('example.metric.2')
[{u'timestamp': 1429711362289, u'value': 3.45}, {u'timestamp': 1429711311895, u'value': 2.345}]
```

To push multiple metrics with multiple values per metric, see metrics_test.py and method ``test_add_multi_metrics_and_datapoints()``.

### Querying metric values

Querying metrics is limited to just metric_id querying now with given search_options. Search for tagged data is coming up later. The supported parameters are ``start``, ``end``, ``buckets``, ``bucketDuration`` and ``distinct``. The returned data structure will change depending on the given parameters. 

```python
>>> c.query_metric(MetricType.Gauge, 'test.query.numeric.1')
[{u'timestamp': 1429816731689, u'value': 1.45}, {u'timestamp': 1429816729689, u'value': 2.0}]
>>> c.query_metric(MetricType.Gauge, 'test.query.numeric.1', start=1429816731689)
[{u'timestamp': 1429816731689, u'value': 1.45}]
>>> c.query_metric(MetricType.Gauge, 'test.query.numeric.1', buckets=1)
[{u'end': 1429816997651, u'min': 1.45, u'max': 2.0, u'median': 1.725, u'value': u'NaN', u'start': 1429788197651, u'avg': 1.725, u'empty': False, u'percentile95th': 2.0}]
```

## Method documentation

Method documentation is available with ``pydoc hawkular``
