# Keecker Remote Messaging

- Chose a transport:
  - [Bluetooth](Bluetooth Transport.md)
  - [WiFi / Ethernet](WebSocket Transport.md)
- Exchange Protocol Buffers messages between your companion app and Keecker.
- Start by sending and receiving encryption keys.
- Encrypt the following messages with NaCl.

# Python implementation

The following snippet uses the `libnacl` and `protobuf` (3.0.0b2) dependencies.

Start by compiling the messages to Python, you can find them in the [./proto](../proto) folder.

```bash
protoc --proto_path ./proto/ --python_out=messages/ ./proto/*
```

```python
from libnacl.public import Box

import importlib
from messages.KeeckerMessage_pb2 import KeeckerMessage, AuthMessage

class KeeckerMessaging:

    def __init__(self, adapter):
        self.adapter = adapter
        self.box = None

    def exchange_keys(self, public_key, private_key):
        msg = AuthMessage()
        msg.key = public_key
        msg.userId = "no-longer-used@keecker.com"
        msg.deviceId = "Python Remote"
        self.send(msg)
        msg = self.recv()
        if len(msg.key) != len(private_key):
            print("Bad key ! Keecker most likely refused your connection")
        else:
            print('Connection is now secured')
            self.box = Box(private_key, msg.key)

    def send(self, message):
        wrapper = KeeckerMessage()
        wrapper.type_name = message.DESCRIPTOR.full_name.replace(
            "com.keecker.common.messages.", "")
        wrapper.message = message.SerializeToString()
        message = wrapper.SerializeToString()
        if self.box != None:
            message = self.box.encrypt(message)
        self.adapter.send(message)

    def recv(self):
        wrapper = KeeckerMessage()
        msg = self.adapter.recv()
        if self.box != None:
            msg = self.box.decrypt(result)
        wrapper.ParseFromString(msg)
        # Instanciate message from type
        module = importlib.import_module("messages.KeeckerMessage_pb2")
        class_ = getattr(module, wrapper.type_name)
        message = class_()
        message.ParseFromString(wrapper.message)
        return message

```
