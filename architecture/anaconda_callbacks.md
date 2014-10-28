# Anaconda Callbacks

Anaconda implements a simple error safe non retriable callbacks mechanism that offer some convenience features to naked function calls. Instances of the `Callback` class that lives in `anaconda_lib.callback` module can be passed to asynchronous client methods, that is, can be passed as first argument to the `execute` method of the `Worker` instances.

`Callback` instances is the recommended method to use within anaconda GUI commands and listener hooks so it's highly convenient for you to get familiar with them.

## How they work?

A `Callback` instance can contain three different callables that will be fired when certain events occurs. Those events are:

1. `on_sucess`: the callable assigned to it will be fired on success calls
2. `on_failure`: the callable assigned to it will be fired on failed calls
3. `on_timeout`: the callable assigned to it will be fired on timeouts

<blockquote>**note**: a callback object can be fired only once, try to fire an already fired callback more than once results in a `RuntimeError` raising.</blockquote>

The only mandatory action is the `on_success` meaning that if you don't define callbacks for the `on_failure` and `on_tieout` actions, the `on_success` one will be fired in case of timeouts or failures and this may not be what you want to do.

## Define a callback

Callback actions can be defined in the `Callback` constructor when you are instanciating a new `Callback` object or after than using the convenience method `on`. The constructor also accepts a `timeout` parameter that will set the timeout for the `on_timeout` firing (in seconds) that by default is `0`

```python
...
# instanciate a new configured Callback object
callback = Callback(
    on_succes=lambda data: pass,
    on_failure=lambda data: pass,
    on_timeout=lambda data: pass
)
Worker.execute(callback, **data)
...
```

<blockquote>**note**: if the `timeout` in a callback is set to `0` or less, the `on_timeout` will never be fired.</blockquote>

Anaconda's `Callback` also implement a convenience `on` method that you can use to assign actions afetr the `Callback` object has been created

```python
...
callback = Callback(timeout=1)
callback.on(success=get_data)
callback.on(error=show_error)
callback.on(timeout=show_error)
...
```

<blockquote>**note**: in the case of using the `on` method, the failure is mapped as `error`</blockquote>

The last useful method in the `Callback` object is the `timeout` property (that is a `@property` method) that can be used to assign a timeout to a callback after it has been created (or to retrive it's value).

```python
...
callback = Callback()
callback.timeout = 2
...
```

<blockquote>**remmeber**: all the timeouts are in seconds, so if you want to assign a timeout of half a second, use the decimal notation e.g. `callback.timeout = 0.5`</blockquote>
