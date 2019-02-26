# Keecker Bluetooth Transport

- Connect to Keecker with the RFCOMM Bluetooth protocol.
- Use the `7fdc2820-6bc9-11e3-981f-0800200c9a66` service UUID.
- Sent and receive messages preceded by their size encoded in plain ASCII, like `10\x00..........`.

## Pyhton implementation

The following snippet uses the `pybluez` dependency.

```python
from bluetooth import BluetoothSocket, RFCOMM, find_service

class BluetoothTransport:

    KEECKER_SERVICE_UUID = "7fdc2820-6bc9-11e3-981f-0800200c9a66"

    def connect(self, address = None):
        service_matches = find_service(
            uuid = self.KEECKER_SERVICE_UUID, address = None)
        if len(service_matches) == 0:
            print("Couldn't find the Keecker service")
            return False
        service = (service_matches[0]["host"], service_matches[0]["port"])
        print("Connecting to {}".format(service))
        self.sock=BluetoothSocket(RFCOMM)
        self.sock.connect(service)
        return True

    def send(self, data):
        # Sends data size first
        self.sock.send(str(len(data)) + "\0")
        # Then actual data
        self.sock.send(data)

    def recv(self):
        # Receives data size first
        size = 0
        while True:
            c = self.sock.recv(1)
            if c == "\0":
                break
            else:
                size = size * 10 + int(c)
        # Then actual data
        data = ""
        while len(data) < size:
            data += self.sock.recv(size - len(data))
        return data

    def close(self):
        self.sock.close()

```
