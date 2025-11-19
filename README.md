# MoodyLib.GameStateManager

A lightweight stack-based game state machine for Unity.  
Useful for UI flows, gameplay mode switching, pause systems, cutscenes, and any scenario where states need to be layered and resumed.

## Contents
- **AbstractGameStateManager** – Base class that manages a stack of states and handles state transitions.
- **IGameStateManager** – Interface for interacting with any game state manager implementation.
- **IGameState** – Interface representing an enterable/exitable game state.
- **UiGameState** – A simple built-in implementation for toggling UI GameObjects when entering or exiting a state.

## Install via Git URL

1. In Unity, open **Window > Package Manager**.
2. Click the **+** button in the top-left corner.
3. Select **Add package from Git URL…**
4. Paste the following URL and click **Add**:

   ```text
   https://github.com/fapoli/MoodyLib.GameStateManager.git
   ```

Unity will download and install the package. After installation, it will appear under the **Packages** folder.

## How to use

### 1. Create a state
Create a class that implements `IGameState`:

```csharp
public class MainMenuState : IGameState {
    public void OnStateEnter() {
        Debug.Log("Entered main menu");
    }

    public void OnStateExit() {
        Debug.Log("Exited main menu");
    }
}
```

### 2. Create your manager
Inherit from `AbstractGameStateManager` and define the initial state:

```csharp
public class MyGameManager : AbstractGameStateManager {
    protected override IGameState GetInitialState() {
        return new MainMenuState();
    }
}
```

### 3. Push & pop states

From anywhere in your project (or from your states):

```csharp
var gsm = ServiceLocator.Resolve<IGameStateManager>();
gsm.Push(new PauseState());
gsm.Pop();
```

### 4. Use UiGameState for UI flows
Perfect for menus, overlays, HUDs, or panels:

```csharp
public class UiFlowManager : AbstractGameStateManager {
    [SerializeField] private GameObject mainMenu;
    [SerializeField] private GameObject pauseMenu;

    protected override IGameState GetInitialState() {
        return new UiGameState(mainMenu);
    }

    public void Pause() {
        Push(new UiGameState(pauseMenu));
    }

    public void Unpause() {
        Pop();
    }
}
```

The manager automatically shows/hides the UI objects when states transition.

## Best Practices

### ✔ Keep states self-contained  
States should manage their own behavior and not rely on global references.

### ✔ Avoid heavy initialization inside states  
Constructors should be cheap. Use OnStateEnter for activation logic.

### ✔ UI states should not modify gameplay  
Keep UI and gameplay states separate to maintain clarity.

### ✔ Always return a valid initial state  
`GetInitialState()` must return a non-null state instance.

### ✔ Combine with ServiceLocator for decoupled access  
Register your manager:

```csharp
ServiceLocator.Register<IGameStateManager>(this);
```

Then resolve it anywhere:

```csharp
var gsm = ServiceLocator.Resolve<IGameStateManager>();
```

### ✔ Use the stack intentionally  
Push temporary states (pause, inventory, settings), pop them to resume.

## Method Reference

### `Push(IGameState state)`
Adds a new state to the stack.  
Calls `OnStateExit` on the previous state and `OnStateEnter` on the new one.

### `Pop()`
Removes the current state.  
Calls `OnStateExit` on the popped state, then `OnStateEnter` on the new top state (if any).

### `Current`
Gets the active (topmost) state, or `null` if the stack is empty.

### `IGameState.OnStateEnter()`
Called when a state becomes active.

### `IGameState.OnStateExit()`
Called when a state stops being active.

Refer to the XML documentation inside each class for detailed information about available methods and parameters.
