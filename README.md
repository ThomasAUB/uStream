[![Build Status](https://app.travis-ci.com/ThomasAUB/uStream.svg)](https://travis-ci.com/ThomasAUB/uStream)
[![License](https://img.shields.io/github/license/ThomasAUB/uStream.svg)](LICENSE)

# uStream

uStream is a lightweight dependency injection tool meant for microcontrollers.
It uses ID to differentiate streams, a stream can support multiple sources and multiple types.
A destination can be connected and disconnected from a stream.

## How to

First, you'll have to define at least one stream ID

Example :

```cpp
enum class eMyStreams {
    eStream
};
```

Then you can define free or static functions to receive the streams

```cpp
struct MyArgType1 {/**/};
struct MyArgType2 {/**/};

struct RX1 {

    static bool input(MyArgType1) {
        /**/
    }

};

struct RX2 {

    template<typename T>
    static bool process(const MyArgType1, T&) {
        /**/
    }

};
```

Now it's time to associate this class to the stream somewhere in your code. A mutable stream can be changed afterwards and an immutable can't. You can set several functions on the same stream as long as the prototypes are not the same

```cpp
// set RX1::input as the mutable function called on MyStreams::eStream
ustream::Channel<eMyStreams::eStream>::setMutable(&RX1::input);

// set RX1::input as the immutable function called on MyStreams::eStream
ustream::Channel<eMyStreams::eStream>::setImmutable(&RX2::process<MyArgType2>);
```

Now, you can start to use the stream

```cpp
#include "ustream.h"

int main() {

    // instantiate a socket returning bool and taking MyArgType1 in argument
    ustream::Socket<bool(MyArgType1)> socket1;

    // attach the socket to eMyStreams::eStream
    socket1.attach<eMyStreams::eStream>();

    // send data on the stream
    socket1(MyArgType1());

    // instantiate a socket returning bool and taking const MyArgType1 and 
    // MyArgType2& in argument
    ustream::Socket<bool(const MyArgType1, MyArgType2&)> socket2;

    // attach the socket to eMyStreams::eStream
    socket2.attach<eMyStreams::eStream>();

    MyArgType2 arg;

    // send data on the stream
    socket2(MyArgType1(), arg);

    return 0;
}
```

It's also possible to send callbacks on a stream

```cpp
static void asyncProcess(int i, void(*inCallback)(int)) {
    /*...*/
    inCallback(42); 
}
```

```cpp
// instantiate socket
ustream::Socket<void(int, void(*)(int))> asyncSocket;

/*...*/
asyncSocket.attach<eMyStreams::eStream>();

ustream::Channel<eMyStreams::eStream>::setMutable(asyncProcess);
/*...*/

// use the socket with a lambda expression callback
asyncSocket(5, [](int inResult) 
{
    if(inResult == 42) {
        /*...*/
    }
}

);

```

The main interest of the Socket object is to attach out of the scope of usage, if the ID of a channel is known in the scope of usage, you can use the call function
```cpp
static void receive(int i) {
    /*...*/
}

#include "ustream.h"
void init() {
    // attach a function tot the stream somewhere in the code
    ustream::Channel<eMyStreams::eStream>::setMutable(receive);
}
```
```cpp
#include "ustream.h"

int main() {
    // calls the "receive" function
    ustream::Channel<eMyStreams::eStream>::call(456);
}
```
It's possible to change the channel function at run time if declared as mutable
```cpp
static void receive2(int i) {
    /*...*/
}
static void receive1(int i) {
    /*...*/
    ustream::Channel<eMyStreams::eStream>::setMutable(receive2);
}

#include "ustream.h"
void init() {
    // attach a function tot the stream somewhere in the code
    ustream::Channel<eMyStreams::eStream>::setMutable(receive1);
}
```
```cpp
#include "ustream.h"

int main() {
    // calls the "receive1" function
    ustream::Channel<eMyStreams::eStream>::call(456);
    
    // calls the "receive2" function
    ustream::Channel<eMyStreams::eStream>::call(456);
}
```
