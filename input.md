
# Input Handling

Responsive input is crucial for great gameplay. VoidRay provides a comprehensive input system that handles keyboard, mouse, and gamepad input with frame-perfect detection and customizable controls.

## 🎮 Input System Overview

The input system provides:
- **Keyboard Input**: All keys with press/hold/release detection
- **Mouse Input**: Position, buttons, wheel with world/screen coordinates
- **Gamepad Support**: Multiple controllers with analog sticks
- **Frame-Perfect Detection**: Just-pressed and just-released events
- **Input Mapping**: Customizable control schemes

## ⌨️ Keyboard Input

### Basic Key Detection
```python
from voidray import Keys
import voidray

def update(self, delta_time):
    input_mgr = voidray.get_engine().input_manager
    
    # Check if key is currently held down
    if input_mgr.is_key_pressed(Keys.SPACE):
        print("Space is being held")
    
    # Check if key was just pressed this frame
    if input_mgr.is_key_just_pressed(Keys.ENTER):
        print("Enter was just pressed")
    
    # Check if key was just released this frame
    if input_mgr.is_key_just_released(Keys.ESCAPE):
        print("Escape was just released")
```

### Movement Input
```python
class PlayerController(Component):
    def __init__(self):
        super().__init__()
        self.speed = 300
    
    def update(self, delta_time):
        input_mgr = voidray.get_engine().input_manager
        
        # Method 1: Individual key checks
        movement = Vector2.zero()
        
        if input_mgr.is_key_pressed(Keys.W):
            movement.y = -1
        if input_mgr.is_key_pressed(Keys.S):
            movement.y = 1
        if input_mgr.is_key_pressed(Keys.A):
            movement.x = -1
        if input_mgr.is_key_pressed(Keys.D):
            movement.x = 1
        
        # Method 2: Axis-based input
        horizontal = input_mgr.get_axis(Keys.A, Keys.D)  # -1 to 1
        vertical = input_mgr.get_axis(Keys.W, Keys.S)    # -1 to 1
        movement = Vector2(horizontal, -vertical)  # Invert Y for screen coords
        
        # Method 3: Built-in movement vector (WASD + Arrow keys)
        movement = input_mgr.get_movement_vector()
        
        # Apply movement
        if movement.magnitude() > 1:
            movement = movement.normalized()  # Prevent diagonal speed boost
        
        velocity = movement * self.speed
        self.game_object.transform.translate(velocity * delta_time)
```

### Action Input (Jumping, Shooting, etc.)
```python
class ActionController(Component):
    def __init__(self):
        super().__init__()
        self.can_jump = True
        self.fire_rate = 0.2  # Seconds between shots
        self.time_since_shot = 0
    
    def update(self, delta_time):
        input_mgr = voidray.get_engine().input_manager
        self.time_since_shot += delta_time
        
        # Jumping (single press action)
        if input_mgr.is_key_just_pressed(Keys.SPACE) and self.can_jump:
            self.jump()
        
        # Shooting (continuous or single shot)
        if input_mgr.is_key_pressed(Keys.LEFT_CTRL):
            # Continuous shooting with fire rate limit
            if self.time_since_shot >= self.fire_rate:
                self.shoot()
                self.time_since_shot = 0
        
        # Alternative: Single shot per press
        if input_mgr.is_key_just_pressed(Keys.X):
            self.shoot()
    
    def jump(self):
        rigidbody = self.game_object.get_component(Rigidbody)
        if rigidbody:
            rigidbody.add_impulse(Vector2(0, -500))
        self.can_jump = False
    
    def shoot(self):
        # Create bullet logic here
        print("Bang!")
```

### Key Constants Reference
```python
# Letter keys
Keys.A, Keys.B, Keys.C, ..., Keys.Z

# Number keys
Keys.KEY_0, Keys.KEY_1, ..., Keys.KEY_9  # Top row numbers
Keys.NUM_0, Keys.NUM_1, ..., Keys.NUM_9  # Numpad numbers

# Arrow keys
Keys.UP, Keys.DOWN, Keys.LEFT, Keys.RIGHT

# Function keys
Keys.F1, Keys.F2, ..., Keys.F12

# Modifier keys
Keys.SHIFT, Keys.CTRL, Keys.ALT
Keys.LEFT_SHIFT, Keys.RIGHT_SHIFT
Keys.LEFT_CTRL, Keys.RIGHT_CTRL

# Special keys
Keys.SPACE, Keys.ENTER, Keys.ESCAPE, Keys.TAB, Keys.BACKSPACE
Keys.DELETE, Keys.HOME, Keys.END, Keys.PAGE_UP, Keys.PAGE_DOWN
```

## 🖱️ Mouse Input

### Mouse Position and Buttons
```python
def update(self, delta_time):
    input_mgr = voidray.get_engine().input_manager
    
    # Get mouse position (screen coordinates)
    mouse_pos = input_mgr.get_mouse_position()
    print(f"Mouse at: {mouse_pos.x}, {mouse_pos.y}")
    
    # Get mouse position in world coordinates
    camera = voidray.get_engine().camera
    world_pos = input_mgr.get_mouse_world_position(camera)
    print(f"World position: {world_pos.x}, {world_pos.y}")
    
    # Mouse button detection
    if input_mgr.is_mouse_button_pressed(MouseButtons.LEFT):
        print("Left mouse button held")
    
    if input_mgr.is_mouse_button_just_pressed(MouseButtons.RIGHT):
        print("Right mouse button just clicked")
    
    if input_mgr.is_mouse_button_just_released(MouseButtons.MIDDLE):
        print("Middle mouse button just released")
    
    # Mouse wheel
    wheel_delta = input_mgr.get_mouse_wheel_delta()
    if wheel_delta != 0:
        print(f"Mouse wheel: {wheel_delta}")  # Positive = up, negative = down
```

### Mouse-Based Controls
```python
class MouseController(Component):
    def __init__(self):
        super().__init__()
        self.drag_start = None
        self.is_dragging = False
    
    def update(self, delta_time):
        input_mgr = voidray.get_engine().input_manager
        mouse_pos = input_mgr.get_mouse_position()
        
        # Click to move
        if input_mgr.is_mouse_button_just_pressed(MouseButtons.LEFT):
            self.move_to_mouse(mouse_pos)
        
        # Drag object
        if input_mgr.is_mouse_button_just_pressed(MouseButtons.RIGHT):
            self.start_drag(mouse_pos)
        elif input_mgr.is_mouse_button_pressed(MouseButtons.RIGHT) and self.is_dragging:
            self.update_drag(mouse_pos)
        elif input_mgr.is_mouse_button_just_released(MouseButtons.RIGHT):
            self.end_drag()
        
        # Zoom with mouse wheel
        wheel_delta = input_mgr.get_mouse_wheel_delta()
        if wheel_delta != 0:
            self.zoom_camera(wheel_delta)
    
    def move_to_mouse(self, target_position):
        # Convert screen to world position
        camera = voidray.get_engine().camera
        world_target = camera.screen_to_world(target_position)
        
        # Move towards target
        direction = (world_target - self.game_object.transform.position).normalized()
        rigidbody = self.game_object.get_component(Rigidbody)
        if rigidbody:
            rigidbody.add_impulse(direction * 300)
    
    def start_drag(self, mouse_pos):
        self.is_dragging = True
        self.drag_start = mouse_pos
    
    def update_drag(self, mouse_pos):
        if self.drag_start:
            # Calculate drag offset
            offset = mouse_pos - self.drag_start
            camera = voidray.get_engine().camera
            world_offset = camera.screen_to_world(offset) - camera.screen_to_world(Vector2.zero())
            
            # Apply to object
            self.game_object.transform.translate(world_offset)
            self.drag_start = mouse_pos
    
    def end_drag(self):
        self.is_dragging = False
        self.drag_start = None
    
    def zoom_camera(self, wheel_delta):
        camera = voidray.get_engine().camera
        zoom_factor = 1.1 if wheel_delta > 0 else 0.9
        camera.set_zoom(camera.zoom * zoom_factor)
```

### Click Detection on Objects
```python
class ClickableObject(Sprite):
    def __init__(self, name):
        super().__init__(name)
        self.create_colored_rect(50, 50, Color.BLUE)
        
        # Add collider for click detection
        self.collider = BoxCollider(50, 50)
        self.add_component(self.collider)
    
    def update(self, delta_time):
        super().update(delta_time)
        
        input_mgr = voidray.get_engine().input_manager
        
        if input_mgr.is_mouse_button_just_pressed(MouseButtons.LEFT):
            if self.is_mouse_over():
                self.on_click()
    
    def is_mouse_over(self):
        """Check if mouse is over this object"""
        input_mgr = voidray.get_engine().input_manager
        camera = voidray.get_engine().camera
        
        mouse_world_pos = input_mgr.get_mouse_world_position(camera)
        obj_pos = self.transform.position
        
        # Simple bounding box check
        half_width = 25
        half_height = 25
        
        return (abs(mouse_world_pos.x - obj_pos.x) <= half_width and
                abs(mouse_world_pos.y - obj_pos.y) <= half_height)
    
    def on_click(self):
        print(f"{self.name} was clicked!")
        # Change color or trigger action
        self.color = Color.RED
```

## 🎮 Gamepad Support

### Gamepad Input
```python
def update(self, delta_time):
    input_mgr = voidray.get_engine().input_manager
    
    # Check if gamepad is connected
    if not input_mgr.is_gamepad_connected(0):  # Player 1
        return
    
    # Button input
    if input_mgr.is_gamepad_button_pressed(0, GamepadButtons.A):
        self.jump()
    
    if input_mgr.is_gamepad_button_just_pressed(0, GamepadButtons.X):
        self.shoot()
    
    # Analog stick input
    left_stick = input_mgr.get_gamepad_stick(0, GamepadStick.LEFT)
    right_stick = input_mgr.get_gamepad_stick(0, GamepadStick.RIGHT)
    
    # Movement with left stick
    movement = left_stick * self.speed * delta_time
    self.game_object.transform.translate(movement)
    
    # Camera control with right stick
    camera = voidray.get_engine().camera
    camera_movement = right_stick * 200 * delta_time
    camera.transform.translate(camera_movement)
    
    # Triggers (0.0 to 1.0)
    left_trigger = input_mgr.get_gamepad_trigger(0, GamepadTrigger.LEFT)
    right_trigger = input_mgr.get_gamepad_trigger(0, GamepadTrigger.RIGHT)
    
    # Use triggers for variable actions
    if right_trigger > 0.1:  # Deadzone
        self.accelerate(right_trigger)
    if left_trigger > 0.1:
        self.brake(left_trigger)
```

## 🔧 Input Mapping System

### Customizable Controls
```python
class InputMapper:
    def __init__(self):
        self.key_bindings = {
            'move_up': [Keys.W, Keys.UP],
            'move_down': [Keys.S, Keys.DOWN],
            'move_left': [Keys.A, Keys.LEFT],
            'move_right': [Keys.D, Keys.RIGHT],
            'jump': [Keys.SPACE],
            'shoot': [Keys.LEFT_CTRL, Keys.X],
            'interact': [Keys.E, Keys.ENTER],
            'pause': [Keys.ESCAPE, Keys.P]
        }
        
        self.mouse_bindings = {
            'aim': MouseButtons.RIGHT,
            'fire': MouseButtons.LEFT,
            'special': MouseButtons.MIDDLE
        }
        
        self.gamepad_bindings = {
            'jump': GamepadButtons.A,
            'shoot': GamepadButtons.X,
            'interact': GamepadButtons.B,
            'pause': GamepadButtons.START
        }
    
    def is_action_pressed(self, action):
        """Check if any key bound to this action is pressed"""
        input_mgr = voidray.get_engine().input_manager
        
        # Check keyboard
        if action in self.key_bindings:
            for key in self.key_bindings[action]:
                if input_mgr.is_key_pressed(key):
                    return True
        
        # Check mouse
        if action in self.mouse_bindings:
            button = self.mouse_bindings[action]
            if input_mgr.is_mouse_button_pressed(button):
                return True
        
        # Check gamepad
        if action in self.gamepad_bindings:
            button = self.gamepad_bindings[action]
            if input_mgr.is_gamepad_button_pressed(0, button):
                return True
        
        return False
    
    def is_action_just_pressed(self, action):
        """Check if any key bound to this action was just pressed"""
        input_mgr = voidray.get_engine().input_manager
        
        # Check keyboard
        if action in self.key_bindings:
            for key in self.key_bindings[action]:
                if input_mgr.is_key_just_pressed(key):
                    return True
        
        # Check mouse
        if action in self.mouse_bindings:
            button = self.mouse_bindings[action]
            if input_mgr.is_mouse_button_just_pressed(button):
                return True
        
        # Check gamepad
        if action in self.gamepad_bindings:
            button = self.gamepad_bindings[action]
            if input_mgr.is_gamepad_button_just_pressed(0, button):
                return True
        
        return False
    
    def bind_key(self, action, key):
        """Bind a new key to an action"""
        if action not in self.key_bindings:
            self.key_bindings[action] = []
        self.key_bindings[action].append(key)
    
    def unbind_key(self, action, key):
        """Remove a key binding"""
        if action in self.key_bindings and key in self.key_bindings[action]:
            self.key_bindings[action].remove(key)

# Usage in your game
input_mapper = InputMapper()

class PlayerController(Component):
    def update(self, delta_time):
        # Use action-based input instead of direct keys
        movement = Vector2.zero()
        
        if input_mapper.is_action_pressed('move_left'):
            movement.x = -1
        if input_mapper.is_action_pressed('move_right'):
            movement.x = 1
        if input_mapper.is_action_pressed('move_up'):
            movement.y = -1
        if input_mapper.is_action_pressed('move_down'):
            movement.y = 1
        
        if input_mapper.is_action_just_pressed('jump'):
            self.jump()
        
        if input_mapper.is_action_just_pressed('shoot'):
            self.shoot()
        
        # Apply movement
        velocity = movement * self.speed * delta_time
        self.game_object.transform.translate(velocity)
```

## 📱 Advanced Input Techniques

### Input Buffering
```python
class InputBuffer:
    def __init__(self, buffer_time=0.1):
        self.buffer_time = buffer_time
        self.buffered_actions = {}
    
    def buffer_action(self, action):
        """Buffer an action for later execution"""
        self.buffered_actions[action] = self.buffer_time
    
    def update(self, delta_time):
        """Update buffer timers"""
        expired_actions = []
        for action, time_left in self.buffered_actions.items():
            time_left -= delta_time
            if time_left <= 0:
                expired_actions.append(action)
            else:
                self.buffered_actions[action] = time_left
        
        for action in expired_actions:
            del self.buffered_actions[action]
    
    def consume_action(self, action):
        """Check if action is buffered and consume it"""
        if action in self.buffered_actions:
            del self.buffered_actions[action]
            return True
        return False

class PlayerController(Component):
    def __init__(self):
        super().__init__()
        self.input_buffer = InputBuffer()
        self.on_ground = False
    
    def update(self, delta_time):
        input_mgr = voidray.get_engine().input_manager
        self.input_buffer.update(delta_time)
        
        # Buffer jump input
        if input_mgr.is_key_just_pressed(Keys.SPACE):
            self.input_buffer.buffer_action('jump')
        
        # Execute buffered jump when landing
        if self.on_ground and self.input_buffer.consume_action('jump'):
            self.jump()
```

### Combo System
```python
class ComboSystem:
    def __init__(self):
        self.combo_window = 0.5  # Time between inputs
        self.input_sequence = []
        self.time_since_last_input = 0
        
        # Define combos
        self.combos = {
            ('punch', 'punch', 'kick'): 'triple_combo',
            ('down', 'forward', 'punch'): 'fireball',
            ('back', 'forward', 'kick'): 'special_move'
        }
    
    def add_input(self, input_action):
        """Add an input to the sequence"""
        self.input_sequence.append(input_action)
        self.time_since_last_input = 0
        
        # Check for combos
        self.check_combos()
        
        # Limit sequence length
        if len(self.input_sequence) > 10:
            self.input_sequence.pop(0)
    
    def update(self, delta_time):
        self.time_since_last_input += delta_time
        
        # Clear sequence if too much time has passed
        if self.time_since_last_input > self.combo_window:
            self.input_sequence.clear()
    
    def check_combos(self):
        """Check if current sequence matches any combos"""
        sequence_tuple = tuple(self.input_sequence)
        
        for combo_sequence, combo_name in self.combos.items():
            if sequence_tuple.endswith(combo_sequence):
                self.execute_combo(combo_name)
                self.input_sequence.clear()
                break
    
    def execute_combo(self, combo_name):
        print(f"Executed combo: {combo_name}")
        # Trigger combo-specific actions
```

### Context-Sensitive Input
```python
class ContextInputManager:
    def __init__(self):
        self.current_context = "game"
        self.context_handlers = {
            "game": self.handle_game_input,
            "menu": self.handle_menu_input,
            "dialogue": self.handle_dialogue_input,
            "inventory": self.handle_inventory_input
        }
    
    def set_context(self, context):
        self.current_context = context
        print(f"Input context changed to: {context}")
    
    def handle_input(self, delta_time):
        handler = self.context_handlers.get(self.current_context)
        if handler:
            handler(delta_time)
    
    def handle_game_input(self, delta_time):
        input_mgr = voidray.get_engine().input_manager
        
        # Normal game controls
        if input_mgr.is_key_pressed(Keys.W):
            # Move player
            pass
        
        if input_mgr.is_key_just_pressed(Keys.E):
            # Check for interactables
            self.try_interact()
    
    def handle_menu_input(self, delta_time):
        input_mgr = voidray.get_engine().input_manager
        
        # Menu navigation
        if input_mgr.is_key_just_pressed(Keys.UP):
            self.navigate_menu(-1)
        if input_mgr.is_key_just_pressed(Keys.DOWN):
            self.navigate_menu(1)
        if input_mgr.is_key_just_pressed(Keys.ENTER):
            self.select_menu_item()
    
    def handle_dialogue_input(self, delta_time):
        input_mgr = voidray.get_engine().input_manager
        
        # Advance dialogue
        if input_mgr.is_key_just_pressed(Keys.SPACE):
            self.advance_dialogue()
        if input_mgr.is_key_just_pressed(Keys.ESCAPE):
            self.skip_dialogue()
```

## 🎯 Input Best Practices

### Performance Optimization
```python
class OptimizedInputHandler:
    def __init__(self):
        # Cache input manager reference
        self.input_mgr = voidray.get_engine().input_manager
        
        # Cache frequently used keys
        self.movement_keys = [Keys.W, Keys.A, Keys.S, Keys.D]
        self.action_keys = [Keys.SPACE, Keys.E, Keys.LEFT_CTRL]
    
    def update(self, delta_time):
        # Batch input checks
        pressed_keys = []
        just_pressed_keys = []
        
        for key in self.movement_keys + self.action_keys:
            if self.input_mgr.is_key_pressed(key):
                pressed_keys.append(key)
            if self.input_mgr.is_key_just_pressed(key):
                just_pressed_keys.append(key)
        
        # Process results
        self.handle_pressed_keys(pressed_keys, delta_time)
        self.handle_just_pressed_keys(just_pressed_keys)
```

### Accessibility Features
```python
class AccessibleInput:
    def __init__(self):
        self.hold_to_toggle = False  # Hold keys act as toggles
        self.key_repeat_delay = 0.5  # Delay before key repeat
        self.key_repeat_rate = 0.1   # Rate of key repeat
        self.sticky_keys = False     # Modifier keys stick
    
    def is_key_active(self, key):
        """Check if key is active with accessibility features"""
        input_mgr = voidray.get_engine().input_manager
        
        if self.hold_to_toggle:
            # Implement toggle logic
            return self.check_toggle_state(key)
        else:
            return input_mgr.is_key_pressed(key)
    
    def enable_colorblind_support(self):
        """Provide alternative feedback for colorblind players"""
        # Use patterns, shapes, or sounds instead of color
        pass
    
    def enable_motor_assistance(self):
        """Provide assistance for players with motor difficulties"""
        # Implement auto-aim, larger hit boxes, etc.
        pass
```

## 📚 Next Steps

Master input handling in VoidRay:

1. **[Physics System](physics.md)** - Combine input with physics movement
2. **[UI System](ui.md)** - Handle UI input and interactions
3. **[Audio System](audio.md)** - Add audio feedback to input
4. **[Examples](examples.md)** - See complete input implementations

Great input handling makes the difference between a frustrating and enjoyable game. Practice these patterns to create responsive, accessible controls! 🎮
