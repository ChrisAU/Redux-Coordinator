# Redux-Coordinator

# Business Logic

## Action

`Actions` describe a change to the `State` they may contain additional information in order to make this change such as results from an API.

`Actions` must adhere to the `ActionType` protocol.

**Note:** In Swift, enums are perfectly suited for defining similar `Actions`.

```
enum FetchAction: ActionType {
    case started
    case success(Posts)
    case failure(Error)
}
```

Combined with generics you may be able to get away with just typealiases in many scenarios.

```
enum LoadAction<T>: ActionType {
    case started
    case success(T)
    case failure(Error)
}
typealias FetchAction = LoadAction<Posts>
```

## ActionCreator

`Action Creators` are exactly that — functions that create `Actions`. It's easy to conflate the terms "action" and "action creator", so do your best to use the proper term.

Asynchronous `ActionCreator`'s can return an initial `Action` (such as loading) followed by other `Actions` as the asynchronous task progresses this can help you notify the user of a percentage of completion, a successful result, and an error.

Consider this _asynchronous_ action creator, it returns a result immediately, followed by another one after the API responds:

```
func fetchPosts(from url: URL) -> ActionType {
    api.fetch(url, method: .GET) { response, error in
        if let error = error {
            store.dispatch(FetchAction.failure(error))
        } else {
            store.dispatch(FetchAction.success(Posts.parse(response))
        }
    }
    return FetchAction.started
}
```

Alternatively, _synchronous_ action creators would likely trigger either action A or B depending on state:

```
func toggleSwitch(_ on: Bool) -> ActionType {
    return on ? SwitchAction.off : SwitchAction.on
}
```

NOTE: `Action Creators` should be used in all cases where something is dispatched to the `Store` from the `Coordinator`.

## Reducer

`Reducers` are *pure* functions, they produce no side-effects and will always return the same output for a given input.

`Reducers` specify how the application's `State` changes in response to `Actions` sent to the `Store`.

`Reducers` can call other reducers in order to update different `State` objects.

**Remember:** `Actions` only describe the fact that something happened, but don't describe _how_ the application's `State` changes.

Each `State` object must define a mutating function as shown below:

```
struct CountState: StateType {
    var counter: Int = 0

    mutating func reduce(_ action: ActionType) {
        switch action {
        case CountAction.increment: counter += 1
        case CountAction.decrement: counter -= 1
        default: break
        }
    }
}
```

## Middleware

The `Middleware` provides a third-party extension point between dispatching an `Action`, and the moment it reaches the `Reducer`. People use Redux middleware for logging, crash reporting, talking to an asynchronous API, routing, and more.

## State

The `State` is a representation of state for a given context. The context could be the application, a screen, a route, and many more.

The `State` object is immutable, it is a representation of the _current_ state.

In Redux there is only one application `State`. However, inside that object there might be other `SubState` objects that are updated by `Reducers`.

## Store

The `Store` is responsible for dispatching `Actions` to the `Middlewares` and main `Reducer` to mutate the `State`.

The `Store` holds onto the latest application `State`.

The `Store` provides methods of observing changes to `State` via `observe(keyPath)`, `observeOnMain(keyPath)`.

**Important:** you'll only have a single store in a Redux application.

Observing changes using `KeyPath`:
```
store.observe(\.counter).subscribe { newCount in
    print("New count is:", newCount)
}
```

# UI

## Coordinator

Responsible for navigation, calling the `Renderer` when `State` changes, and receiving `Actions` from the `View`.

`Actions` may be perform routing here to achieve deep linking (`RouteAction`).

## Renderer

Responsible for rendering the `ViewState` on the `View`. 

This is typically done with an extension on the `ViewController`.

## ViewState

In order to present something meaningful in the `View` we need to map the `State` into something more consumable. An example of this would be turning a raw number (e.g. `10`) into a piece of text like `Seconds elapsed: 10`.

## View

The `ViewController` should be split into three parts to clearly compartmentalise the parts into:
- VC (Triggers `Actions`, holds `viewDidLoad` logic which may call `Layout` code)
- VC+Layout (Handles Constraint Changes)
- VC+Renderer (Handles `ViewState` Changes)
