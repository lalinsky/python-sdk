# Infinario Python SDK

![](https://travis-ci.org/Infinario/python-sdk.svg)

The `infinario.Infinario` class provides access to the Infinario Python tracking API,
supporting both synchronous and asynchronous modes.
In order to track events, instantiate the class at least with your project token
(can be found in Project Management in your Infinario account), for example:

```python
from infinario import Infinario

client = Infinario('12345678-90ab-cdef-1234-567890abcdef')                  # PRODUCTION ENVIRONMENT
# client = Infinario('12345678-90ab-cdef-1234-567890abcdef', silent=False)  # DEVELOPMENT ENVIRONMENT
```

We recommend to set the `silent` parameter to `False` in a development environment, as it will cause the Infinario API
to throw exceptions if something goes wrong. When left to the default value `True`, all errors will be logged
(also see the `logger` parameter).


## Identifying the customer

When tracking events, you have to specify which customer generated
them. This can be either done right when calling the client's
constructor.

```python
client = Infinario('12345678-90ab-cdef-1234-567890abcdef', customer='john123')
```

or by calling `identify`.

```python
client.identify('john123')
```

## Tracking events

To track events for the currently selected customer, simply
call the `track` method.

```python
client.track('purchase')
```

You can also specify a dictionary of event properties to store
with the event.

```python
client.track('purchase', {'product': 'bottle', 'amount': 5})
```

Specify the POSIX timestamp of the event using:

```python
timestamp = time.time()

# .. time passes ..

client.track('purchase', timestamp=timestamp)
```

## Updating customer properties

You can also update information that is stored with a customer.

```python
client.update({'first_name': 'John', 'last_name': 'Smith'})
```

## Getting HTML from campaign

```python
client.get_html('Banner left')
```

will return

```python
'<img src="/my-awesome-banner-1.png" />'
```

## Accessing analyses

To get results of existing analyses stored in your Infinario project, you need to initialize the client
with the Infinario project secret (found in the Overview screen) as the `secret` keyword argument.

```python
client = Infinario('12345678-90ab-cdef-1234-567890abcdef', customer='john123',
                   secret='fedcba09-8765-4321-fedc-ba0987654321')
```

### Analysis export

To export the entire result of an analysis, use the `export_analysis` client method.
First argument is type of analysis (funnel, report, retention, segmentation), second argument is JSON object
containing at least the ID of the analysis to export.

```python
client.export_analysis('funnel', {
    'analysis_id': '2f86608f-24f5-11e3-9950-c48508494cf5'
})
```

which could return

```python
{
    "success": true,
    "name": "Conversion funnel",
    "steps": ["First visit", "Registration", "First log in", "Purchase", "Payment"],
    "total": {
        "counts": [48632, 24120, 20398, 1256, 1250],
        "times": [-1, 680, 4502, 45, 540, 300],
        "metric": 1987562
    },
    "drill_down": {
        "type": "none",
        "series": []
    },
    "metric": {
        "step": 4,
        "property": "price"
    }
}
```

### Segmentation result

You can also export the result of a segmentation for a specific customer
(whom you need to specify either at initialization, or using the `identify` method).

```python
client.segment_for('11112222-3333-4444-5555-666677778888', timezone='UTC', timeout=0.5)
```

which could return a string like `'Heavy payer'`. In case the customer doesn't belong to any defined segment or
their segmentation could not be determined within the given timeout, the method will return `None`.
The `timezone` and `timeout` parameters are optional with the defaults as in the example.


## Transport types

By default the client uses a simple non-buffered synchronous transport. The three available transport types are:
* `NullTransport` - No requests, useful for disabling tracking in the Infinario constructor.
* `SynchronousTransport` - (default) Most operations are blocking for the time of a request to the Infinario API
* `AsynchronousTransport` - Most operations are non-blocking (see the code for more information),
    buffered and using a single worker thread. Infinario client must be closed when no more data is to be tracked. 
    **We recommend against using the AsynchronousTransport, as it cannot be guaranteed the data will be sent.**
    Data loss can for example happen in various events of system failure or even due to misuse.
    If you would like to track data from your code asynchronously, consider creating your own asynchronous workers
    using a library such as celery and use the SynchronousTransport to send the data from there.

Example of choosing `AsynchronousTransport`:

```python
from infinario import Infinario, AsynchronousTransport

client = Infinario('12345678-90ab-cdef-1234-567890abcdef',
                   transport=AsynchronousTransport)

# ...

client.close()
```


## Using on the command line

The python client also has a command-line interface that allows to call its essential functions.

```bash
TOKEN='12345678-90ab-cdef-1234-567890abcdef'
CUSTOMER='john123'

# Track event
./infinario.py track "$TOKEN" "$CUSTOMER" purchase --properties product=bottle amount=5

# Update customer properties
./infinario.py update "$TOKEN" "$CUSTOMER" first_name=John last_name=Smith

# Get HTML from campaign
./infinario.py get_html "$TOKEN" "$CUSTOMER" "Banner left"
```
