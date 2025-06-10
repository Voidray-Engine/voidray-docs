
# Engine Overview

VoidRay is a modern, component-based 2D/2.5D game engine built with Python. This guide explains the core architecture and design principles that make VoidRay powerful and easy to use.

## 🏗️ Architecture Overview

VoidRay follows a **component-based architecture** similar to Unity, where functionality is added to objects through modular components rather than inheritance.

```
VoidRay Engine
├── Core Systems
│   ├── Engine (Main loop, initialization)
│   ├── Scene Manager (Scene switching, loading)
│   ├── Input Manager (Keyboard, mouse, gamepad)
│   ├── Physics System (Collision, rigidbodies)
│   └── Audio Manager (Music, sound effects)
├── Graphics Pipeline
│   ├── Renderer (Drawing, batching)
│   ├── Camera System (View management)
│   ├── Sprite System (2D graphics)
│   └── Particle System (Visual effects)
├── Game Framework
│   ├── Game Objects (Entities in your game)
│   ├── Components (Modular behaviors)
│   ├── Scenes (Game states/levels)
│   └── Transform System (Position, rotation, scale)
└── Tools & Utilities
    ├── Asset Loader (Resources)
    ├── Debug Overlay (Performance monitoring)
    ├── UI System (Interface elements)
    └── Scripting (Custom behaviors)
```

## 🎮 Core Concepts

### 1. Engine
The **Engine** is the heart of VoidRay. It manages:
- Main game loop (update, render, input)
- System initialization and coordination
- Frame rate management
- Resource allocation

```python
import voidray

# Configure the engine
voidray.configure(800, 600, "My Game", 60)

# Get engine reference
engine = voidray.get_engine()
print(f"FPS: {engine.get_fps()}")
```

### 2. Scenes
**Scenes** are containers for your game content - like levels, menus, or game states:

```python
class MenuScene(Scene):
    def on_enter(self):
        # Initialize menu
        pass
    
    def on_exit(self):
        # Cleanup
        pass
    
    def update(self, delta_time):
        # Menu logic
        pass

# Register and switch scenes
voidray.register_scene("menu", MenuScene())
voidray.set_scene("menu")
```

### 3. Game Objects
**Game Objects** are entities in your game world. Everything visible or interactive is a GameObject:

```python
from voidray import GameObject, Sprite

# Basic game object
player = GameObject("Player")

# Sprite (visual game object)
enemy = Sprite("Enemy")
enemy.create_colored_rect(30, 30, Color.RED)
```

### 4. Components
**Components** add specific behaviors to GameObjects. This modular approach makes objects flexible and reusable:

```python
from voidray import Rigidbody, BoxCollider

# Add physics to a game object
rigidbody = Rigidbody()
rigidbody.set_mass(2.0)
player.add_component(rigidbody)

# Add collision detection
collider = BoxCollider(40, 40)
player.add_component(collider)

# Get components later
rb = player.get_component(Rigidbody)
if rb:
    rb.add_force(Vector2(100, 0))
```

### 5. Transform System
Every GameObject has a **Transform** that controls position, rotation, and scale:

```python
# Position in world space
player.transform.position = Vector2(100, 200)

# Rotation in degrees
player.transform.rotation = 45

# Scale (1.0 = normal size)
player.transform.scale = Vector2(2.0, 1.5)

# Relative movement
player.transform.translate(Vector2(10, 0))
```

## 🔄 Game Loop

VoidRay's game loop follows this pattern every frame:

```
1. Input Processing
   ├── Handle keyboard/mouse events
   ├── Update input state
   └── Process user input

2. Update Phase
   ├── Update current scene
   ├── Update all game objects
   ├── Update physics simulation
   └── Update audio/animation systems

3. Render Phase
   ├── Clear screen
   ├── Render scene objects (sorted by layer)
   ├── Render UI elements
   ├── Apply post-processing effects
   └── Present to screen

4. Frame Management
   ├── Calculate delta time
   ├── Maintain target FPS
   └── Handle frame rate limiting
```

## 🎯 Design Principles

### 1. Component-Based Design
Instead of complex inheritance hierarchies, VoidRay uses composition:

```python
# Bad: Complex inheritance
class FlyingShootingEnemy(Enemy, Flying, Shooter):
    pass

# Good: Component composition
enemy = Sprite("Enemy")
enemy.add_component(EnemyAI())
enemy.add_component(FlyingMovement())
enemy.add_component(Weapon())
```

### 2. Data-Driven Architecture
Game behavior is defined by data, not hard-coded logic:

```python
# Configure behavior with data
enemy_config = {
    'health': 100,
    'speed': 150,
    'weapon_type': 'laser',
    'ai_pattern': 'chase_player'
}

enemy = create_enemy(enemy_config)
```

### 3. Performance-First
VoidRay is designed for smooth 60+ FPS gameplay:
- Object pooling for frequently created/destroyed objects
- Spatial partitioning for collision detection
- Batch rendering for sprites
- Component caching for fast access

### 4. Developer-Friendly
The API is designed to be intuitive and discoverable:
- Clear naming conventions
- Comprehensive error messages
- Built-in debugging tools
- Extensive documentation

## 🚀 System Integration

### Physics Integration
The physics system seamlessly integrates with GameObjects:

```python
# Physics automatically updates transform
rigidbody.add_force(Vector2(100, 0))
# GameObject position updates automatically

# Collision callbacks
def on_hit(other, collision_info):
    print(f"Hit {other.game_object.name}")

collider.on_collision = on_hit
```

### Audio Integration
Audio is spatially aware and integrates with the camera system:

```python
# 2D positioned audio
audio.play_sound_3d("explosion.wav", explosion_position)

# Automatic volume based on camera distance
audio.set_listener_position(camera.position)
```

### Input Integration
Input seamlessly flows through the system:

```python
def update(self, delta_time):
    input_mgr = voidray.get_engine().input_manager
    
    if input_mgr.is_key_just_pressed(Keys.SPACE):
        self.jump()
    
    movement = input_mgr.get_movement_vector()
    self.move(movement * self.speed * delta_time)
```

## 📈 Performance Characteristics

### Memory Management
- **Automatic cleanup** of destroyed objects
- **Resource pooling** for frequently used assets
- **Garbage collection optimization** to minimize frame drops
- **Memory monitoring** with built-in profiler

### Rendering Performance
- **Batch rendering** reduces draw calls
- **Frustum culling** skips off-screen objects
- **Layer-based rendering** for optimal z-ordering
- **Texture atlasing** for efficient GPU usage

### Physics Performance
- **Spatial partitioning** (quadtree) for collision detection
- **Sleeping objects** automatically pause when stationary
- **Broadphase/narrowphase** collision detection
- **Physics timestep** independent of frame rate

## 🛠️ Development Workflow

### 1. Project Setup
```python
# Main game file structure
def init_game():
    # Initialize your scenes
    pass

def main():
    voidray.configure(800, 600, "My Game", 60)
    voidray.on_init(init_game)
    voidray.start()
```

### 2. Scene Development
```python
class GameScene(Scene):
    def on_enter(self):
        # Load level, create objects
        pass
    
    def update(self, delta_time):
        # Game logic
        pass
    
    def render(self, renderer):
        # Custom drawing
        pass
```

### 3. Object Creation
```python
# Create game objects with components
player = create_player()
enemies = create_enemy_wave()
ui = create_game_ui()

# Add to scene
scene.add_object(player)
for enemy in enemies:
    scene.add_object(enemy)
```

### 4. Testing & Debugging
```python
# Enable debug overlay
engine.debug_overlay.enabled = True

# Performance monitoring
stats = engine.get_performance_stats()
print(f"Objects: {stats['object_count']}")
print(f"FPS: {stats['fps']}")
```

## 🎯 Next Steps

Now that you understand VoidRay's architecture:

1. **[Game Objects & Components](game_objects.md)** - Deep dive into the component system
2. **[Scene Management](scenes.md)** - Learn advanced scene techniques
3. **[Physics System](physics.md)** - Master realistic physics
4. **[First Game Tutorial](first_game_tutorial.md)** - Build a complete game

The component-based architecture makes VoidRay incredibly flexible - you can create any type of 2D game while maintaining clean, maintainable code.

Ready to start building? 🚀
