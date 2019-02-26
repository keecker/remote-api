# Keecker WebSocket Transport

- Find the Keecker IP address / Hostname with Zeroconf
- Connect to `ws://<IP/Hostname>:8080/remote/keecker.ws`

## Python implementation

The following snippet uses the `websocket-client` dependency.

```python
from websocket import create_connection

class WebSocketTransport:

    KEECKER_URI = "ws://{}:8080/remote/keecker.ws"

    def connect(self, address):
        self.ws = create_connection(self.KEECKER_URI.format(address))

    def send(self, data):
        self.ws.send_binary(data)

    def recv(self):
        return self.ws.recv()

    def close(self):
        self.ws.close()
```
