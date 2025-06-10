
# Game Objects & Components

Game Objects are the foundation of everything in your VoidRay game. This guide covers the component-based system that makes VoidRay flexible and powerful.

## 🎯 Core Concepts

### Game Objects
A **GameObject** is any entity in your game world - players, enemies, bullets, collectibles, even invisible logic controllers. By themselves, GameObjects don't do much - their power comes from **Components**.

### Components
**Components** are modular pieces of functionality that you attach to GameObjects. Instead of complex inheritance, you compose objects by adding components:

```python
# Create a player
player = Sprite("Player")                    # Visual representation
player.add_component(Rigidbody())           # Physics movement
player.add_component(BoxCollider(40, 40))   # Collision detection
player.add_component(PlayerController())    # Custom behavior
```

## 🏗️ Creating Game Objects

### Basic GameObject
```python
from voidray import GameObject, Vector2

# Create an empty game object
controller = GameObject("GameController")
controller.transform.position = Vector2(0, 0)

# Game objects without visuals are useful for:
# - Game logic controllers
# - Spawn points
# - Trigger zones
# - Audio sources
```

### Sprites (Visual Game Objects)
```python
from voidray import Sprite
from voidray.graphics.renderer import Color

# Create a visual game object
enemy = Sprite("Enemy")

# Method 1: Colored shapes
enemy.create_colored_rect(30, 30, Color.RED)
enemy.create_colored_circle(15, Color.BLUE)

# Method 2: Load from image file
enemy.load_texture("assets/enemy.png")

# Method 3: Create from existing texture
texture = asset_loader.load_texture("player.png")
player = Sprite("Player")
player.texture = texture
```

### Setting Properties
```python
# Position, rotation, scale
enemy.transform.position = Vector2(200, 100)
enemy.transform.rotation = 45  # degrees
enemy.transform.scale = Vector2(2.0, 1.5)  # 2x width, 1.5x height

# Sprite-specific properties
enemy.color = Color.RED        # Tint color
enemy.alpha = 0.8             # Transparency
enemy.flip_x = True           # Mirror horizontally
enemy.layer = 10              # Rendering order
```

## 🧩 Built-in Components

### Transform Component
Every GameObject automatically has a Transform:

```python
# Position (world coordinates)
obj.transform.position = Vector2(100, 200)
obj.transform.translate(Vector2(10, 0))  # Move relative

# Rotation (degrees)
obj.transform.rotation = 90
obj.transform.rotate(45)  # Rotate by amount

# Scale
obj.transform.scale = Vector2(2.0, 2.0)  # Double size

# Hierarchy (parent-child relationships)
child.transform.parent = parent.transform
```

### Rigidbody Component
Adds physics simulation:

```python
from voidray import Rigidbody

rigidbody = Rigidbody()

# Mass affects how forces move the object
rigidbody.set_mass(2.0)  # Heavier = harder to push

# Drag slows down movement (air resistance)
rigidbody.set_drag(0.1)  # 0.0 = no drag, 1.0 = max drag

# Bounciness for collisions
rigidbody.set_bounciness(0.8)  # 0.0 = no bounce, 1.0 = perfect bounce

# Control physics behavior
rigidbody.is_kinematic = False    # Affected by physics
rigidbody.use_gravity = True      # Affected by gravity
rigidbody.freeze_rotation = False # Can rotate

player.add_component(rigidbody)

# Apply forces
rigidbody.add_force(Vector2(100, 0))      # Continuous force
rigidbody.add_impulse(Vector2(0, -500))   # Instant impulse (jump)
rigidbody.set_velocity(Vector2(200, 0))   # Direct velocity control
```

### Collider Components
Enable collision detection:

```python
from voidray import BoxCollider, CircleCollider

# Box collision (rectangles)
box_collider = BoxCollider(40, 30)  # width, height
player.add_component(box_collider)

# Circle collision (circles)
circle_collider = CircleCollider(20)  # radius
ball.add_component(circle_collider)

# Collision callbacks
def on_player_hit(other, collision_info):
    if "Enemy" in other.game_object.name:
        print("Player hit enemy!")
        # Handle collision

box_collider.on_collision = on_player_hit

# Trigger zones (no physics collision, just detection)
trigger = BoxCollider(100, 100)
trigger.is_trigger = True  # Won't block movement
trigger.on_collision = handle_trigger_enter
```

## 🎮 Custom Components

Create your own components for game-specific behavior:

```python
from voidray import Component

class PlayerController(Component):
    def __init__(self):
        super().__init__()
        self.speed = 300
        self.jump_force = 500
        self.health = 100
        self.on_ground = False
    
    def start(self):
        """Called once when component is added"""
        print(f"Player controller started for {self.game_object.name}")
        
        # Get references to other components
        self.rigidbody = self.game_object.get_component(Rigidbody)
        self.collider = self.game_object.get_component(BoxCollider)
        
        if self.collider:
            self.collider.on_collision = self.on_collision
    
    def update(self, delta_time):
        """Called every frame"""
        self.handle_input(delta_time)
        self.check_bounds()
    
    def handle_input(self, delta_time):
        input_mgr = voidray.get_engine().input_manager
        
        # Movement
        move_x = 0
        if input_mgr.is_key_pressed(Keys.LEFT):
            move_x = -1
        if input_mgr.is_key_pressed(Keys.RIGHT):
            move_x = 1
        
        if self.rigidbody:
            # Physics-based movement
            self.rigidbody.add_force(Vector2(move_x * self.speed * 10, 0))
        else:
            # Direct movement
            movement = Vector2(move_x * self.speed * delta_time, 0)
            self.game_object.transform.translate(movement)
        
        # Jumping
        if input_mgr.is_key_just_pressed(Keys.SPACE) and self.on_ground:
            if self.rigidbody:
                self.rigidbody.add_impulse(Vector2(0, -self.jump_force))
            self.on_ground = False
    
    def on_collision(self, other, collision_info):
        """Handle collisions"""
        # Check if landing on ground
        if collision_info.normal.y < 0:  # Collision from above
            self.on_ground = True
        
        # Check for enemies
        if "Enemy" in other.game_object.name:
            self.take_damage(10)
    
    def take_damage(self, amount):
        self.health -= amount
        print(f"Player health: {self.health}")
        
        if self.health <= 0:
            self.game_object.destroy()
    
    def check_bounds(self):
        """Keep player on screen"""
        engine = voidray.get_engine()
        pos = self.game_object.transform.position
        
        # Clamp to screen bounds
        pos.x = max(20, min(engine.width - 20, pos.x))
        pos.y = max(20, min(engine.height - 20, pos.y))

# Use the custom component
player = Sprite("Player")
player.create_colored_rect(40, 40, Color.BLUE)
player.add_component(Rigidbody())
player.add_component(BoxCollider(40, 40))
player.add_component(PlayerController())  # Our custom behavior
```

## 🔍 Component System Patterns

### Getting Components
```python
# Check if object has a component
if player.has_component(Rigidbody):
    print("Player has physics")

# Get a component
rigidbody = player.get_component(Rigidbody)
if rigidbody:
    rigidbody.add_force(Vector2(100, 0))

# Get all components of a type from scene
physics_objects = scene.find_objects_with_component(Rigidbody)
```

### Component Communication
```python
class WeaponComponent(Component):
    def fire(self):
        # Create bullet
        bullet = Sprite("Bullet")
        bullet.transform.position = self.game_object.transform.position
        
        # Add to same scene as this object
        scene = self.game_object.scene
        scene.add_object(bullet)

class PlayerController(Component):
    def update(self, delta_time):
        input_mgr = voidray.get_engine().input_manager
        
        if input_mgr.is_key_just_pressed(Keys.SPACE):
            # Get weapon component and fire
            weapon = self.game_object.get_component(WeaponComponent)
            if weapon:
                weapon.fire()
```

### Component Dependencies
```python
class HealthComponent(Component):
    def __init__(self, max_health=100):
        super().__init__()
        self.max_health = max_health
        self.current_health = max_health
    
    def take_damage(self, amount):
        self.current_health = max(0, self.current_health - amount)
        if self.current_health <= 0:
            self.die()
    
    def die(self):
        # Notify other components
        death_component = self.game_object.get_component(DeathComponent)
        if death_component:
            death_component.handle_death()
        
        self.game_object.destroy()

class DeathComponent(Component):
    def handle_death(self):
        # Play death animation, sound effects, etc.
        print(f"{self.game_object.name} has died!")
```

## 🎯 Complete Examples

### Simple Enemy
```python
class SimpleEnemy(Sprite):
    def __init__(self):
        super().__init__("Enemy")
        self.create_colored_rect(30, 30, Color.RED)
        
        # Add physics
        rigidbody = Rigidbody()
        rigidbody.set_mass(1.0)
        self.add_component(rigidbody)
        
        # Add collision
        collider = BoxCollider(30, 30)
        collider.on_collision = self.on_collision
        self.add_component(collider)
        
        # Add custom behavior
        self.add_component(EnemyAI())
    
    def on_collision(self, other, collision_info):
        if "Player" in other.game_object.name:
            print("Enemy hit player!")

class EnemyAI(Component):
    def __init__(self):
        super().__init__()
        self.speed = 100
        self.target = None
    
    def start(self):
        # Find player
        self.target = self.game_object.scene.find_object("Player")
    
    def update(self, delta_time):
        if not self.target:
            return
        
        # Move toward player
        direction = (self.target.transform.position - 
                    self.game_object.transform.position).normalized()
        
        rigidbody = self.game_object.get_component(Rigidbody)
        if rigidbody:
            rigidbody.add_force(direction * self.speed * 10)
```

### Collectible Item
```python
class Collectible(Sprite):
    def __init__(self, item_type="coin", value=10):
        super().__init__(f"Collectible_{item_type}")
        self.create_colored_circle(12, Color.YELLOW)
        
        self.item_type = item_type
        self.value = value
        
        # Trigger collision (no physics blocking)
        collider = CircleCollider(12)
        collider.is_trigger = True
        collider.on_collision = self.on_collect
        self.add_component(collider)
        
        # Add animation
        self.add_component(FloatAnimation())
    
    def on_collect(self, other, collision_info):
        if "Player" in other.game_object.name:
            # Give player the item
            player_controller = other.game_object.get_component(PlayerController)
            if player_controller:
                player_controller.collect_item(self.item_type, self.value)
            
            # Remove this collectible
            self.destroy()

class FloatAnimation(Component):
    def __init__(self):
        super().__init__()
        self.time = 0
        self.start_y = 0
        self.float_speed = 2
        self.float_amount = 10
    
    def start(self):
        self.start_y = self.game_object.transform.position.y
    
    def update(self, delta_time):
        self.time += delta_time
        
        # Floating motion
        offset = math.sin(self.time * self.float_speed) * self.float_amount
        self.game_object.transform.position.y = self.start_y + offset
        
        # Rotation
        self.game_object.transform.rotation += 90 * delta_time
```

## 🚀 Best Practices

### 1. Single Responsibility
Each component should have one clear purpose:
```python
# Good: Focused components
class MovementComponent(Component): pass
class HealthComponent(Component): pass
class WeaponComponent(Component): pass

# Bad: Monolithic component
class PlayerEverythingComponent(Component): pass
```

### 2. Component Communication
Use interfaces for clean communication:
```python
class IDamageable:
    def take_damage(self, amount): pass

class HealthComponent(Component, IDamageable):
    def take_damage(self, amount):
        self.health -= amount

# Other components can damage anything that implements IDamageable
```

### 3. Performance Considerations
```python
# Cache component references
class PlayerController(Component):
    def start(self):
        self.rigidbody = self.game_object.get_component(Rigidbody)
        self.collider = self.game_object.get_component(BoxCollider)
    
    def update(self, delta_time):
        # Use cached references instead of getting components each frame
        if self.rigidbody:
            self.rigidbody.add_force(force)
```

### 4. Component Lifecycle
```python
class MyComponent(Component):
    def start(self):
        """Called once when component is added"""
        pass
    
    def update(self, delta_time):
        """Called every frame if enabled"""
        pass
    
    def on_destroy(self):
        """Called when component/object is destroyed"""
        # Cleanup resources
        pass
```

## 📚 Next Steps

Now that you understand the component system:

1. **[Physics System](physics.md)** - Deep dive into Rigidbody and Collider components
2. **[Scene Management](scenes.md)** - Learn how to organize objects in scenes
3. **[Input Handling](input.md)** - Create responsive player controllers
4. **[Examples](examples.md)** - See complete game object implementations

The component system is VoidRay's superpower - master it and you can create any game object you can imagine! 🎮
