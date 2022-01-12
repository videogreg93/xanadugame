---

tags: "LibGDX LibKTX"

---

Enabling remappable controls in your game is as simple as decoupling the player's input from the actual action that they want to preform with said input. Let's start by taking a look at a standard implementation of handling input in **LibGDX**, with the help of [LibKTX](https://github.com/libktx/ktx).

```kotlin
class MenuScreen: KtxScreen, KtxInputAdapter {
  override fun show() {
      super.show()
      val inputProcessor = Gdx.input.inputProcessor
      val multiplexer = InputMultiplexer(this, inputProcessor)
      Gdx.input.inputProcessor = multiplexer
  }
  
  override fun keyDown(keycode: Int): Boolean {
      when(keycode) {
        Input.Keys.E -> {
          // Do Something when 'e' key is pressed
        }
        else -> return false
      }
    return true
  }
  
  override fun hide() {
      super.hide()
      (Gdx.input.inputProcessor as? InputMultiplexer)?.removeProcessor(this)
  }
}
```

While this might be sufficient for a simple game, hardcoded key inputs into a screen gives us hardly any flexibility. If for example we wanted to replace all actions currently handled with the 'e' key by the 'q' key, we'd have to go over all of our screens and replace it. Even worse, this doesn't allow users to remap keys themselves, since we've tightly coupled our desired action to the 'se key. 

How about if instead of listening to keys, we listened to actions instead? Let's replace `keyDown` with `onAction` and turn it into something like this :

```kotlin
override fun onAction(action: Action): Boolean {
      when(action) {
        INTERACT -> {
          // handle interact action
        }
        else -> return false
      }
  return true
  }
```

Let's see how to get something like this up and running.

### Creating our own InputProcessor

We're going to need to place something between LibGDX's input processor and our screens to intercept user input and replace it with our actions. We'll call this `InputActionManager`. 

```kotlin
class InputActionManager : KtxInputAdapter {

    fun init() {
        Gdx.input.inputProcessor = this
    }
  
  enum class InputAction {
        UP,
        DOWN,
        LEFT,
        RIGHT,
        INTERACT,
    }
}
```

Screens won't bind to the `Gdx.input.inputProcessor` anymore. Instead, the sole `InputAdapter` will be this class. Later on, we'll add the screens as listeners of this class to be able to receive events.

We've also added an `InputAction` enum, which represent the different actions we want to handle.

### Mapping Inputs to our Actions

Let's add a hash map to map Input to Actions :

```kotlin
private val keyboardMappings = HashMap<Int, InputAction>().apply {
        put(Input.Keys.UP, InputAction.UP)
        put(Input.Keys.DOWN, InputAction.DOWN)
        put(Input.Keys.LEFT, InputAction.LEFT)
        put(Input.Keys.RIGHT, InputAction.RIGHT)
        put(Input.Keys.E, InputAction.INTERACT)
    }
```

Now we can map our input to our actions whenever the user presses an input.

```kotlin
override fun keyDown(keycode: Int): Boolean {
    val action = keyboardMappings[keycode]
    // Notify listeners of action
}
```

### Introducing the Action Listener

We need a way to notify screens of actions. We'll create an interface called `ActionListener` which screens will be able to implement if they want to handle input.

```kotlin
interface ActionListener {
    fun onAction(action: InputAction): Boolean
}
```

The `InputActionManager` will keep a reference to all `ActionListeners` who have subscribed, and propagate the `Action` until an `ActionListener` handles the action.

```kotlin
class InputActionManager : KtxInputAdapter {
  private val actionListeners = ArrayList<ActionListener>()
  
  private val keyboardMappings = HashMap<Int, InputAction>().apply {
        put(Input.Keys.UP, InputAction.UP)
        put(Input.Keys.DOWN, InputAction.DOWN)
        put(Input.Keys.LEFT, InputAction.LEFT)
        put(Input.Keys.RIGHT, InputAction.RIGHT)
        put(Input.Keys.E, InputAction.INTERACT)
  }
  
  fun subscribe(listener: ActionListener) {
      actionListeners.add(0, listener) // Latest subscribers get priority
  }
  
  fun unsubscribe(listener: ActionListener) {
      actionListeners.remove(listener)
  }
  
  override fun keyDown(keycode: Int): Boolean {
      val action = keyboardMappings[keycode]
      // Notify listeners of action
      action?.let {
          if (listener.onAction(action)) return true
      } ?: debug { "No Action mapped to $keycode" }
      return false
  }
}
```

We've added methods for adding and removing subscribers, and we've created the missing link for notifying subscribers of user actions. We iterate over all subscribers, until we find one which handles the corresponding action. This lets us have multiple classes handling different inputs without the need for a multiplexer. I also added a debug message to notify the dev if they press a key that has no mapped `Action`.

### Implementing an Action Listener

The final piece of the puzzle : implementing an `ActionListener` and having it subscribe to our new `InputActionManager`. You're going to need a way for your objects to access your `InputActionManager`. Here I'm using [KTX-Inject](https://github.com/libktx/ktx/tree/master/inject) to inject it via dependancy injection, but you could also have it as a static object if you prefer.

```kotlin
class MenuScreen: KtxScreen, ActionListener {
  private val inputActionManager: InputActionManager = MainContext.inject()
  override fun show() {
      super.show()
      inputActionManager.subscribe(this)
  }
  
  override fun onAction(action: InputAction): Boolean {
      when(action) {
        INTERACT -> {
          // Handle interact action
        }
        else -> return false
      }
    return true
  }
  
  override fun hide() {
      super.hide()
      inputActionManager.unsubscribe(this)
  }
}
```

Like with the input processor, we must subscribe when we want to receive actions, and unsubscribe when we're done handling actions. in the `onAction` method, we can handle the actions we want, being sure to return `true` when we want to consume an action and `false` when we want to keep propagating the action to let others potentially handle it.

### So What was the Point of it all?

By decoupling our input we've done many things. We now have one central point for input, instead of it being distributed everywhere. That lets us do neat things like disabling all or certain actions if need be, by adding logic in our `InputActionManager` to keep a list of disabled actions and avoid propagating them. We've also made it much easier to implement control remapping. Instead of hardcoding our `keyboardMappings` hash map, we could save that information to a local file, and add an option screen which lets us modify it.

Adding controller support will be much easier, since listeners don't need to listen to both keyboard events and controller events. Just have your `inputActionManager` listen to controller input events and map those to the same Actions as before, that way your listeners don't need to be made aware of the actual input device, just the desired action.

Do you want to show a demo screen where the player is automatically controlled? While out of scope for this article, here's the general idea :

- Hide the inputActionManager implementation behind an interface.
- Create a new implementation of this interface
  - This implementation will propage actions in a certain order, instead of mapping inputs to actions
- Inject this new implementation in your "Demo Screen" instead of the default implementation

You could even replace the demo `InputActionManager` by the real one when the player presses an input, letting them take control at any moment during the demo!

### Things to Improve

Here are a couple of things you could add to make the `InputActionManager` more feature complete :

- Add the possibility to disable certain or all inputs
- Let listeners listen to both `onKeyDown` and `onKeyUp`
- Add controller support