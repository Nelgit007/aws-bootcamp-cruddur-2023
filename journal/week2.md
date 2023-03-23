# Week 2 — Distributed Tracing

## HoneyComb

I set up my development environment to use HONEYCOMB

I added the following Env Vars to the `backend-flask` in docker compose

```yml
OTEL_SERVICE_NAME: "${HONEYCOMB_SERVICE_NAME}"
```
The above instructions would:

export and persist the HONEYCOMB API KEY and OTEL Service name in ENV

**NB: In the event you need traces for multiple backend services its best practice to set the OTEL service for each backend service. The reason is telemetry for software system wise should be treated specific to a service.

I configured OTEL to send data to Honeycomb

```yml
OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
```

I followed the [documentation](https://docs.honeycomb.io/quickstart/#step-3-instrument-your-application-to-send-telemetry-data-to-honeycomb) to install open telemetry for python flask app:

I added the following files to my requirements.txt file

```
opentelemetry-api 
opentelemetry-sdk 
opentelemetry-exporter-otlp-proto-http 
opentelemetry-instrumentation-flask 
opentelemetry-instrumentation-requests
```

I installed the above dependencies:

```sh
pip install -r requirements.txt
```

I added the following lines of code to the `app.py`

```py
from opentelemetry import trace
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
```


```py
# Initialize tracing and an exporter that can send data to Honeycomb
provider = TracerProvider()
processor = BatchSpanProcessor(OTLPSpanExporter())
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)
tracer = trace.get_tracer(__name__)
```

```py
# Initialize automatic instrumentation with Flask
app = Flask(__name__)
FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()
```

I hard-codded a span to the homePageService in backend flask, to  create one.

I followed the OpenTelemetry for Python doccumentation to create a span for the `/api/activities/home` service:

[docs:](https://docs.honeycomb.io/getting-data-in/opentelemetry/python/)

I added Atrributes to spans following the docs:

```py
from opentelemetry import trace

with tracer.start_as_current_span("home-activities-mock-date") as outer_span:
```

I added more Attributes to the span:
```py
span = trace.get_current_span()
span.set_attribute("app.now", now.isoformat)

span.set_attribute("app.result_lenght", len(results))
```

## X-Ray

### Instrumenting AWS X-Ray for Backend-Flask

I followed the [guide](https://github.com/aws/aws-xray-sdk-python) in the AWS-XRAY-SDK:

i added the aws-xray-sdk to the `requirements.txt`

```py
aws-xray-sdk
```

Install the python dendencies for the aws-xray-sdk with:

```sh
pip install -r requirements.txt
```

I added the aws-xray-sdk Recorder and Malware to `app.py`

```py
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware

xray_url = os.getenv("AWS_XRAY_URL")
xray_recorder.configure(service='Cruddur', dynamic_naming=xray_url)
XRayMiddleware(app, xray_recorder)
```

### Setup the AWS X-Ray Resources

I added `aws/json/xray.json`

```json
{
  "SamplingRule": {
      "RuleName": "Cruddur",
      "ResourceARN": "*",
      "Priority": 9000,
      "FixedRate": 0.1,
      "ReservoirSize": 5,
      "ServiceName": "backend-flask",
      "ServiceType": "*",
      "Host": "*",
      "HTTPMethod": "*",
      "URLPath": "*",
      "Version": 1
  }
}
```

I created an X-Ray Group
```sh

aws xray create-group \
   --group-name "Cruddur" \
   --filter-expression "service(\"backend-flask\")"
```

I created a sampling rule.



