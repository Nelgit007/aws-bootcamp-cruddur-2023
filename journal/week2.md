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

Here is a sample Telemetry from Honeycom:
![Traces]()


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
```sh
aws xray create-sampling-rule --cli-input-json file://aws/json/xray.json
```

### Installing X-Ray Daemon

[Install X-ray Daemon](https://docs.aws.amazon.com/xray/latest/devguide/xray-daemon.html)

[Github aws-xray-daemon](https://github.com/aws/aws-xray-daemon)
[X-Ray Docker Compose example](https://github.com/marjamis/xray/blob/master/docker-compose.yml)


### I Added the Daemon Service to Docker Compose

```yml
  xray-daemon:
    image: "amazon/aws-xray-daemon"
    environment:
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
      AWS_REGION: "us-east-1"
    command:
      - "xray -o -b xray-daemon:2000"
    ports:
      - 2000:2000/udp
```

I added the two env vars below to the backend-flask in the `docker-compose.yml` file

```yml
      AWS_XRAY_URL: "*4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}*"
      AWS_XRAY_DAEMON_ADDRESS: "xray-daemon:2000"
```

### Creating Traces in aws-x-ray:

### I added a custom segment/sub-segment to the home_activities services

I followed the openTEL with aws-x-ray SDK python guide:

I imported the x-ray recorder to my `notifications_actitivites` service:

```py
from aws_xray_sdk.core import xray_recorder
```

Using the default begin/end function for implicit recording, i addded this code to my `notifications_activities` service

```py
# Start a segment
segment = xray_recorder.begin_segment('segment_name')

# Start a subsegment
subsegment = xray_recorder.begin_subsegment('subsegment_name')
```

I added a dictionary for metadata:
```py
x_ray_dict = {
  "now": now.isoformat()
  }
```


## CloudWatch Logs


I added the log management service: CloudWatch to the `requirements.txt`

```
watchtower
```

I installed the package using:
```sh
pip install -r requirements.txt
```

Importing the watctower logging in ti `app.py`:

```py
import watchtower
import logging
from time import strftime
`

I added the configuration to Logger to Use CloudWatch
```py
LOGGER = logging.getLogger(__name__)
LOGGER.setLevel(logging.DEBUG)
console_handler = logging.StreamHandler()
cw_handler = watchtower.CloudWatchLogHandler(log_group='cruddur')
LOGGER.addHandler(console_handler)
LOGGER.addHandler(cw_handler)
LOGGER.info("testing Logs in CW")
```

Logging in an Error using cloudwatch logs:

```py
@app.after_request
def after_request(response):
    timestamp = strftime('[%Y-%b-%d %H:%M]')
    LOGGER.error('%s %s %s %s %s %s', timestamp, request.remote_addr, request.method, request.scheme, request.full_path, response.status)
    return response
```

Logging some data in an API endpoint
```py
LOGGER.info('testing Logs in CW! from  /api/activities/notifications')
```

I Set the env var in my your backend-flask for `docker-compose.yml` file.

```yml
      AWS_DEFAULT_REGION: "${AWS_DEFAULT_REGION}"
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
```

## Rollbar

website: https://rollbar.com/

New project `Cruddur`

Adding the libraries to `requirements.txt`


```
blinker
rollbar
```
I set up my access token for rollbar in env

```sh
export ROLLBAR_ACCESS_TOKEN=""
gp env ROLLBAR_ACCESS_TOKEN=""
```

Add to backend-flask for `docker-compose.yml`

```yml
ROLLBAR_ACCESS_TOKEN: "${ROLLBAR_ACCESS_TOKEN}"
```
I added the access token to my `docker-compose.yaml`

```yml
ROLLBAR_ACCESS_TOKEN: "${ROLLBAR_ACCESS_TOKEN}"
```

I imported for Rollbar

```py
import rollbar
import rollbar.contrib.flask
from flask import got_request_exception
```

```py
rollbar_access_token = os.getenv('ROLLBAR_ACCESS_TOKEN')
@app.before_first_request
def init_rollbar():
    """init rollbar module"""
    rollbar.init(
        # access token
        rollbar_access_token,
        # environment name
        'production',
        # server root directory, makes tracebacks prettier
        root=os.path.dirname(os.path.realpath(__file__)),
        # flask already sets up logging
        allow_logging_basic_config=False)

    # send exceptions from `app` to rollbar, using flask's signal system.
    got_request_exception.connect(rollbar.contrib.flask.report_exception, app)
```

Added a new endpoint to test rollbar in `app.py`

```py
@app.route('/rollbar/test')
def rollbar_test():
    rollbar.report_message('Hello World!', 'warning')
    return "Hello World!"
```
Notes: [Rollbar Flask Example](https://github.com/rollbar/rollbar-flask-example/blob/master/hello.py)
