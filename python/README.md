# Mercurio Python SDK

This package is a thin Python client for the Mercurio local HTTP backend.

```python
from mercurio import Mercurio

with Mercurio.launch() as backend:
    workspace = backend.open_workspace("C:/models/demo")
    result = workspace.compile_project()
    graph = workspace.graph()
```

Attach to an already-running backend:

```python
from mercurio import Mercurio

backend = Mercurio.connect("http://127.0.0.1:49152")
workspace = backend.open_workspace("C:/models/demo")
```

The first release expects a Mercurio executable installed separately. Discovery order:

1. Explicit `executable=` argument.
2. `MERCURIO_EXE` environment variable.
3. `mercurio` on `PATH`.
4. Future bundled executable in the Python wheel.

## Capability Authoring

The package also includes `mercurio_capability`, a small SDK for authoring
process-provider capabilities. A capability reads one JSON request from stdin
and writes a `ReasoningCapabilityRunResponse` JSON object to stdout.

Minimal capability:

```python
from mercurio_capability import CapabilityRequest, CapabilityRunner, Finding, ReasoningReport


@CapabilityRunner.capability(
    id="org.example.hello",
    kind="mercurio.capability.kind/static-analysis",
    name="Hello Capability",
    input_artifact_kinds=["kir"],
    output_artifact_kinds=["reasoning_report"],
)
def analyze(request: CapabilityRequest) -> ReasoningReport:
    return request.report_passed(
        findings=[
            Finding.info(
                "hello.ok",
                "Capability ran",
                f"Request {request.request_id} was handled.",
            )
        ]
    )


if __name__ == "__main__":
    CapabilityRunner.run(analyze)
```

Project plugin manifest excerpt:

```json
{
  "capabilities": [
    {
      "capability": {
        "id": "org.example.hello",
        "kind": "mercurio.capability.kind/static-analysis",
        "name": "Hello Capability",
        "version": "0.1.0",
        "api_version": "0.1",
        "deterministic": true,
        "input_artifact_kinds": ["kir"],
        "output_artifact_kinds": ["reasoning_report"]
      },
      "provider": {
        "kind": "process",
        "command": ["python", "plugins/hello/analyze.py"]
      }
    }
  ]
}
```

`CapabilityRunner` validates the descriptor, request, and emitted report shape
before writing stdout. Capability exceptions are returned as structured transport
errors so the Mercurio host can surface a clear failure.
