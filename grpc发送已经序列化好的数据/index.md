# GRPC发送已经序列化好的数据


## 场景

在之前的 gRPC 系统中加了个 redis 缓存临时缓冲要上报的数据，对于数据反复序列化肯定是有性能损耗的

而且 gRPC 提供的接口似乎没有可以直接发送已经序列化好的数据，似乎只能从生成的 gRPC 代码下手

## 实现

以 Python 代码举例：

发现在 `xxx_pb2_grpc.py` 文件中无论是 `Stub` 还是 `Master` 的消息参数中有下面两项，使用默认值 `None` 的话就可以直接传递序列化好的字节了

* request_deserializer：An optional :term:`deserializer` for request deserialization

* response_serializer：An optional :term:`serializer` for response serialization

为了不直接修改 `xxx_pb2_grpc.py`，于是继承 `Stub` 重写 `__init__` 好了

```python
class WithoutSerializerStub(xxx_pb2_grpc.Stub):
    """Missing associated documentation comment in .proto file."""

    def __init__(self, channel):
        """Constructor.

        Args:
            channel: A grpc.Channel.
        """
        self.msg = channel.stream_unary(
                '/server/msg',
                request_serializer=None,
                response_deserializer=None,
                )
```

成功搞定！

## PS

后来尝试继承 `Stub` 后新增一个 `without_serializer` 的 msg，后发现运行时报错找不到这个属性

先记录一下，以后在找原因
