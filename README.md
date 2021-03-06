libsig
======

Signal/Event handling lib for C/C++

[![Build Status](https://travis-ci.org/recp/libsig.svg?branch=master)](https://travis-ci.org/recp/libsig)

ligsig is a signal/event library for C and C++ which allows to handling
signals/events like observer pattern. Member functions also
can be used as callbacks.

Each signal has a signal context which allows that signals can be used in their own
 context.
Also the system signals such as SIGPIPE... can be observed in system context
(sig_sys_ctx) ).

Predefined contexts:
```C
// sig_attach/sig_detach/sig_fire functions
// use the default ctx if there is no specified one
const sig_context_t * sig_ctx_default();

// sistem signals can be observed by using this ctx
const sig_context_t * sig_ctx_sys();
```

Alloc and free custom contexts:
```C
const sig_context_t * sig_ctx_new();
void sig_ctx_free(const sig_context_t * ctx);
```

####For C:####
The functions are overloaded by `_s` postfix (and `c` postfix for custom contexts):
```C
/*
  For all (or full) declerations (especially for custom ctx)
  look at the sig.h header
 */

/* Observe a signal by signal name or id */
void sig_attach(int signal, sig_observer_cb_t cb);
void sig_attach_s(const char * signal, sig_observer_cb_t cb);

/* Stop observe a signal by signal name or id */
void sig_detach(int signal, sig_observer_cb_t cb);
void sig_detach_s(const char * signal, sig_observer_cb_t cb);

/* Fire a signal by signal name or id */
void sig_fire(int signal, void * object);
void sig_fire_s(const char * signal, void * object);
```

####For C++:####
```C++
/*
  For all (or full) declerations (especially for custom ctx)
  look at the sig.h header
 */

/* Observe a signal by signal name or id */
void sig_attach(int signal, sig_observer_cb_t cb);
void sig_attach(const char * signal, sig_observer_cb_t cb);

// For member functions
void sig_attach(int signal, sig_slot(T *, Func));
void sig_attach(const char * signal, sig_slot(T *, Func));

// -------------------------------------------------------

/* Stop observe a signal by signal name or id */
void sig_detach(int signal, sig_observer_cb_t cb);
void sig_detach(const char * signal, sig_observer_cb_t cb);

// For member functions
void sig_detach(int signal, sig_slot(T *, Func));
void sig_detach(const char * signal, sig_slot(T *, Func));

// Detach all observers from the object/instance
void sig_detach(void * observer);
void sig_detach(void * observer, const sig_context_t * ctx);

// -------------------------------------------------------

/* Fire a signal by signal name or id */
void sig_fire(int signal, void * object);
void sig_fire(const char * signal, void * object);
```

Signals also can be observed or can be fired by the following style for C++:

```C++
/* Observe a signal */
sig::attach[1234] << fn_callback1 ... ;
sig::attach["signal_name"] << fn_callback1 << fn_callback2 ...;

/* Stop observe */
sig::detach[1234] >> fn_callback1 ...;
sig::detach["signal_name"] >> fn_callback1 >> fn_callback2 ...;

/* Fire (trigger) a signal */
sig:fire["signal_name"] << (void *)"Signal object (void *)";
```

## Build

### Unix / Macintosh

```text
$ sh autogen.sh
$ ./configure
$ make
$ [sudo] make install
```

### Windows

```text
$ msbuild libsig.vcxproj /p:Configuration=Release
```

###Sample###

```C++
#include <iostream>
#include <signal.h>
#include <sig.h>

#define CUSTOM_NTF1 123

class TestClass {
public:
  TestClass() {
    sig_attach(CUSTOM_NTF1,
               sig_slot(this, &TestClass::memberFn));

    sig_attach("custom_ntf2",
               sig_slot(this, &TestClass::memberFn));

    // Attach / Observe system signals
    sig_attach(SIGUSR1,
               sig_slot(this, &TestClass::memberFn),
               sig_ctx_sys());
  }

  void memberFn(const sig_signal_t sig) {

    // Do somethings...

    std::cout << "notification (TestClass): "
              << (const char *)sig.object
              << std::endl;
  }

  void stopObserveNtf1() {
    sig_detach(CUSTOM_NTF1,
               sig_slot(this, &TestClass::memberFn));
  }

  ~TestClass() {
    sig_detach(this);
  }
};

// non-member function
void do_somethings(const sig_signal_t signal) {
    std::cout << "notification (do_somethings): "
              << (const char *)signal.object
              << std::endl;
}

int main() {

  // Observe a signal/event
  sig_attach("signal-1", do_somethings);

  // Fire signal
  sig_fire("signal-1", (void *)"signal-1 object");


  // Test Member functions

  TestClass t1;

  sig_fire(CUSTOM_NTF1, (void *)"ntf 1 object");
  t1.stopObserveNtf1();
  sig_fire(CUSTOM_NTF1, (void *)"ntf 1 object");

  sig_fire("custom_ntf2", (void *)"ntf 2 object");

  raise(SIGUSR1);

  return 0;
}
```
Output:
```text
notification (do_somethings): signal-1 object
notification (TestClass): ntf 1 object
notification (TestClass): ntf 2 object
notification (TestClass): 
```
