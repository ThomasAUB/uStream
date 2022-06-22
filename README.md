# uStream

uStream is an abstraction tool for communications.
It uses ID to differentiate streams, a stream can support multiple sources, multiple destinations and multiple types.
A destination can be connected and disconnected from a stream.

## How to

First, you'll have to define at least one stream ID.

Example :

```cpp
enum class eMyStreams {
    eStream1,
    eStream2
};
```

Then you can create a class designed to receive a stream.
Once instantiated, the object is **attached**, you can call the **attach()/detach()** functions to enable/disable the stream reception.

```cpp
#include "ustream_input.h"

struct MyInput : ustream::InputBase<MyInput> {

    void uStreamWrite(int a, int b, int c) {
        // do something
    }

    void uStreamRead(float& d) {
        d = 5.48;
    }

};
```

Now it's time to associate this class to the stream somewhere in your code.
Note that you can add as many inputs you want in this macro

```cpp
#include "ustream_router.h"

USTREAM_SET_INPUT(eMyStreams::eStream1, MyInput)
```

Now, you can start to send data on the stream.
Note that you can send any type you want as long as the inputs are able to receive them.

```cpp
#include "ustream_output.h"

int main() {

    // instantiate stream input
    MyInput mi;

    // enable the input
    // note : we could have put this line inside the constructor of "MyInput"
    mi.attach();

    // send an integer rvalue
    ustream::write<eMyStreams::eStream1>(5, 8, 6);

    float f = 0.0;

    // send a float lvalue
    ustream::read<eMyStreams::eStream1>(f);

    // f is now equal to 5.48

}
```

It's also possible to receive a stream in a static constexpr function, in that case you don't have to inherit from InputBase and the function will always be attached.

```cpp
struct MyInput {

    static constexpr bool s_uStreamRead(int& d) {
        d = 66;
        return true;
    }

};
```

Example check :
```cpp
static constexpr int getCTValue() {
    int a = 0;
    ustream::read<eMyStreams::eStream1>(a);
    return a;
}

static_assert(getCTValue() == 66, "Compile-time read fail");
```
If you want to instantiate several time a class that inherits from InputBase and be able to attach and detach them independently, you'll have to explicitely declare the number of instances.


```cpp
struct SeveralInstance : InputBase<SeveralInstance> {
    void uStreamRead(int a) {

    }
}
SeveralInstance A;
SeveralInstance B;
```
```cpp
#include "ustream_input.h"

USTREAM_INPUT_COUNT(SeveralInstance, 2)
```