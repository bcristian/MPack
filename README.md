MPack   
=====
This library is a lightweight implementation of the [MessagePack](http://msgpack.org/) binary serialization format. MessagePack is a 1-to-1 binary representation of JSON, and the official specification can be found here: [https://github.com/msgpack...](https://github.com/msgpack/msgpack/blob/master/spec.md).

[![Build status](https://ci.appveyor.com/api/projects/status/d5d1ry9a53x6vaqu?svg=true)](https://ci.appveyor.com/project/bcristian/mpack)

Notes
-----
* This is my fork of https://github.com/caesay/MPack
* The original serialized all unsigned integers as UInt64, and had a few more issues with integer ranges and conversions.
* The issues did not cause data loss or corruption, but meant that sometimes more bytes than needed were used, and shape checking (e.g. as part of a protocol validation) was hindered.
* Major points from the original:
    * Its easiest to understand how this library works if you think in terms of json. The type `MPackMap` represents a dictionary, and the type `MPackArray` represents an array. 
    * Create MPack instances with the static method `MPack.From(object);`. You can pass any simple type (such as string, integer, etc), or any Array composed of a simple type. MPack also has implicit conversions from most of the basic types built in.
    * Transform an MPack object back into a CLR type with the static method `MPack.To<T>();` or MPack.To(type);. MPack also has **explicit** converions going back to most basic types, you can do `string str = (string)mpack;` for instance.
    * MPack now supports native asynchrounous reading and cancellation tokens. It will *not* block a thread to wait on a stream.

NuGet
-----
This version is not published on nuget.

Usage
-----
Create a object model that can be represented as MsgPack. Here we are creating a dictionary, but really it can be anything:
```csharp
MPackMap dictionary = new MPackMap
{
    {
        "array1", MPack.From(new[]
        {
            "array1_value1",  // implicitly converted string
            MPack.From("array1_value2"),
            MPack.FromString("array1_value3"),
        })
    },
    {"bool1", MPack.From(true)}, //boolean
    {"double1", MPack.From(50.5)}, //single-precision float
    {"double2", MPack.From(15.2)},
    {"int1", 50505}, // implicitly converted integer
    {"int2", MPack.From(50)} // integer
};
```
Serialize the data to a byte array or to a stream to be saved, transmitted, etc:
```csharp
byte[] encodedBytes = dictionary.EncodeToBytes();
// -- or --
dictionary.EncodeToStream(stream);
```
Parse the binary data back into a MPack object model (you can also cast back to an MPackMap or MPackArray after reading if you want dictionary/array methods):
```csharp
var reconstructed = MPack.ParseFromBytes(encodedBytes);
// -- or --
var reconstructed = MPack.ParseFromStream(stream);
```
Turn MPack objects back into types that we understand with the generic `To<>()` method. Since we know the types of everything here we can just call `To<bool>()` to reconstruct our bool, but if you don't know you can access the instance enum `MPack.ValueType` to know what kind of value it is:
```csharp
bool bool1 = reconstructed["bool1"].To<bool>();
var array1 = reconstructed["array1"] as MPackArray;
var array1_value1 = array1[0];
double double1 = reconstructed["double1"].To<double>();
//etc...
```

Credits
-------
The following people/projects have made this possible:

0. Caelan Sayler, the one I forked from: [caelantsayler]at[gmail]dot[com]
0. All of the people that make MessagePack happen: https://github.com/msgpack
0. Me
