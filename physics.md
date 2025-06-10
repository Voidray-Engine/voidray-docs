
# Physics System

VoidRay includes a powerful physics engine that brings your games to life with realistic movement, collision detection, and physics interactions. This guide covers everything from basic movement to advanced physics techniques.

## 🌟 Physics Overview

The physics system consists of several key components:
- **Rigidbody**: Adds physics simulation to objects
- **Colliders**: Define collision shapes and detection
- **Physics System**: Manages gravity, forces, and simulation
- **Materials**: Control friction, bounciness, and other properties

## ⚡ Getting Started with Physics

### Basic Physics Setup
```python
import voidray
from voidray import Sprite, Rigidbody, BoxCollider, Vector2
from voidray.graphics.renderer import Color

class PhysicsObject(Sprite):
    def __init__(self, name):
        super().__init__(name)
        self.create_colored_rect(40, 40, Color.BLUE)
        
        # Add physics simulation
        self.rigidbody = Rigidbody()
        self.rigidbody.set_mass(1.0)
        self.add_component(self.rigidbody)
        
        # Add collision detection
        self.collider = BoxCollider(40, 40)
        self.add_component(self.collider)

# Enable gravity in your scene
def init_game():
    engine = voidray.get_engine()
    engine.physics_system.set_gravity(Vector2(0, 800))  # Downward gravity
```

### Understanding Gravity
```python
# Set global gravity (affects all rigidbodies)
physics_system.set_gravity(Vector2(0, 981))  # Earth-like gravity

# Different gravity effects
physics_system.set_gravity(Vector2(0, 0))     # Zero gravity (space)
physics_system.set_gravity(Vector2(0, 400))   # Light gravity (moon)
physics_system.set_gravity(Vector2(0, 1500))  # Heavy gravity
physics_system.set_gravity(Vector2(-100, 0))  # Side gravity

# Per-object gravity control
rigidbody.gravity_scale = 0.5  # Half gravity
rigidbody.gravity_scale = 0.0  # No gravity
rigidbody.gravity_scale = 2.0  # Double gravity
```

## 🏗️ Rigidbody Component

The Rigidbody component adds physics simulation to GameObjects.

### Basic Properties
```python
rigidbody = Rigidbody()

# Mass affects how forces move the object
rigidbody.set_mass(1.0)     # Default mass
rigidbody.set_mass(0.5)     # Lighter (easier to push)
rigidbody.set_mass(5.0)     # Heavier (harder to push)

# Drag simulates air resistance
rigidbody.set_drag(0.0)     # No air resistance
rigidbody.set_drag(0.1)     # Light air resistance
rigidbody.set_drag(1.0)     # Heavy air resistance

# Angular drag affects rotation
rigidbody.angular_drag = 0.1

# Control physics behavior
rigidbody.is_kinematic = False    # Affected by physics
rigidbody.use_gravity = True      # Affected by gravity
rigidbody.freeze_rotation = False # Can rotate freely
```

### Movement with Forces
```python
# Continuous forces (like thrust or wind)
rigidbody.add_force(Vector2(100, 0))        # Push right
rigidbody.add_force(Vector2(0, -200))       # Push up
rigidbody.add_force(Vector2(-50, 100))      # Push down-left

# Impulses (instant forces like jumping or explosions)
rigidbody.add_impulse(Vector2(0, -500))     # Jump
rigidbody.add_impulse(Vector2(300, -200))   # Jump forward

# Direct velocity control
rigidbody.set_velocity(Vector2(200, 0))     # Move right at 200 units/sec
rigidbody.set_velocity(Vector2(0, 0))       # Stop immediately

# Rotational forces
rigidbody.add_torque(100)                   # Spin clockwise
rigidbody.add_torque(-100)                  # Spin counter-clockwise
```

### Advanced Rigidbody Features
```python
# Bounciness (restitution)
rigidbody.set_bounciness(0.0)   # No bounce (clay)
rigidbody.set_bounciness(0.5)   # Medium bounce
rigidbody.set_bounciness(0.9)   # Very bouncy (rubber ball)
rigidbody.set_bounciness(1.0)   # Perfect bounce

# Constraints
rigidbody.freeze_position_x = True   # Can't move horizontally
rigidbody.freeze_position_y = True   # Can't move vertically
rigidbody.freeze_rotation = True     # Can't rotate

# Kinematic objects (not affected by physics, but can affect others)
rigidbody.is_kinematic = True
# Useful for: platforms, moving obstacles, player controllers
```

## 🔲 Collision Detection

### Collider Types
```python
# Box colliders (rectangles)
box_collider = BoxCollider(width=40, height=30)
player.add_component(box_collider)

# Circle colliders (circles)
circle_collider = CircleCollider(radius=20)
ball.add_component(circle_collider)

# Collider properties
collider.offset = Vector2(5, 0)    # Offset from object center
collider.is_trigger = False         # Physical collision
collider.is_trigger = True          # Trigger zone (no physics block)
```

### Collision Callbacks
```python
class Player(Sprite):
    def __init__(self):
        super().__init__("Player")
        self.create_colored_rect(40, 40, Color.BLUE)
        
        # Add collision detection
        self.collider = BoxCollider(40, 40)
        self.collider.on_collision = self.on_collision
        self.add_component(self.collider)
    
    def on_collision(self, other, collision_info):
        """Called when this object collides with another"""
        other_object = other.game_object
        
        # Check what we hit
        if "Enemy" in other_object.name:
            print("Hit an enemy!")
            self.take_damage(10)
        
        elif "Powerup" in other_object.name:
            print("Collected powerup!")
            other_object.destroy()
        
        elif "Ground" in other_object.name:
            # Check if we landed on top
            if collision_info.normal.y < 0:  # Normal points up
                self.on_ground = True
                print("Landed on ground")
        
        # Use collision info for advanced responses
        collision_point = collision_info.point
        collision_normal = collision_info.normal
        collision_depth = collision_info.depth
```

### Trigger Zones
```python
class TriggerZone(Sprite):
    def __init__(self, width, height):
        super().__init__("TriggerZone")
        # Make invisible (or use a transparent color)
        self.create_colored_rect(width, height, Color(0, 255, 0, 100))
        
        # Create trigger collider
        self.collider = BoxCollider(width, height)
        self.collider.is_trigger = True  # No physical collision
        self.collider.on_collision = self.on_trigger
        self.add_component(self.collider)
    
    def on_trigger(self, other, collision_info):
        if "Player" in other.game_object.name:
            print("Player entered trigger zone!")
            # Trigger events: level changes, cutscenes, etc.
```

## 🎮 Physics Movement Patterns

### Platformer Character Controller
```python
class PlatformerController(Component):
    def __init__(self):
        super().__init__()
        self.speed = 300
        self.jump_force = 600
        self.on_ground = False
        self.coyote_time = 0.1  # Grace period for jumping
        self.time_since_grounded = 0
    
    def start(self):
        self.rigidbody = self.game_object.get_component(Rigidbody)
        self.collider = self.game_object.get_component(BoxCollider)
        if self.collider:
            self.collider.on_collision = self.on_collision
    
    def update(self, delta_time):
        self.handle_movement(delta_time)
        self.handle_jumping(delta_time)
        
        # Update coyote time
        if self.on_ground:
            self.time_since_grounded = 0
        else:
            self.time_since_grounded += delta_time
    
    def handle_movement(self, delta_time):
        input_mgr = voidray.get_engine().input_manager
        
        # Get horizontal input
        horizontal = 0
        if input_mgr.is_key_pressed(Keys.LEFT) or input_mgr.is_key_pressed(Keys.A):
            horizontal = -1
        if input_mgr.is_key_pressed(Keys.RIGHT) or input_mgr.is_key_pressed(Keys.D):
            horizontal = 1
        
        # Apply horizontal force
        if self.rigidbody:
            # Use force for physics-based movement
            force = Vector2(horizontal * self.speed * 10, 0)
            self.rigidbody.add_force(force)
            
            # Limit horizontal speed
            velocity = self.rigidbody.velocity
            max_speed = self.speed
            if abs(velocity.x) > max_speed:
                velocity.x = max_speed if velocity.x > 0 else -max_speed
                self.rigidbody.set_velocity(velocity)
    
    def handle_jumping(self, delta_time):
        input_mgr = voidray.get_engine().input_manager
        
        # Jump with coyote time
        can_jump = self.time_since_grounded <= self.coyote_time
        
        if input_mgr.is_key_just_pressed(Keys.SPACE) and can_jump:
            if self.rigidbody:
                # Set vertical velocity to jump force
                velocity = self.rigidbody.velocity
                velocity.y = -self.jump_force
                self.rigidbody.set_velocity(velocity)
            
            self.on_ground = False
            self.time_since_grounded = self.coyote_time + 1  # Prevent double jump
    
    def on_collision(self, other, collision_info):
        # Check if we're landing on something
        if collision_info.normal.y < -0.7:  # Normal pointing mostly up
            self.on_ground = True
```

### Top-Down Movement
```python
class TopDownController(Component):
    def __init__(self):
        super().__init__()
        self.speed = 250
        self.acceleration = 1000
        self.deceleration = 800
    
    def start(self):
        self.rigidbody = self.game_object.get_component(Rigidbody)
        # Disable gravity for top-down games
        if self.rigidbody:
            self.rigidbody.use_gravity = False
    
    def update(self, delta_time):
        input_mgr = voidray.get_engine().input_manager
        
        # Get input direction
        input_vector = Vector2.zero()
        if input_mgr.is_key_pressed(Keys.W):
            input_vector.y = -1
        if input_mgr.is_key_pressed(Keys.S):
            input_vector.y = 1
        if input_mgr.is_key_pressed(Keys.A):
            input_vector.x = -1
        if input_mgr.is_key_pressed(Keys.D):
            input_vector.x = 1
        
        # Normalize for diagonal movement
        if input_vector.magnitude() > 1:
            input_vector = input_vector.normalized()
        
        if self.rigidbody:
            current_velocity = self.rigidbody.velocity
            target_velocity = input_vector * self.speed
            
            # Smooth acceleration/deceleration
            if input_vector.magnitude() > 0:
                # Accelerating
                new_velocity = current_velocity.lerp(target_velocity, 
                                                   self.acceleration * delta_time / self.speed)
            else:
                # Decelerating
                new_velocity = current_velocity.lerp(Vector2.zero(), 
                                                   self.deceleration * delta_time / self.speed)
            
            self.rigidbody.set_velocity(new_velocity)
```

### Car Physics
```python
class CarController(Component):
    def __init__(self):
        super().__init__()
        self.max_speed = 400
        self.acceleration = 600
        self.turn_speed = 180  # degrees per second
        self.drift_factor = 0.9
    
    def start(self):
        self.rigidbody = self.game_object.get_component(Rigidbody)
        if self.rigidbody:
            self.rigidbody.use_gravity = False
            self.rigidbody.set_drag(2.0)  # Natural slowdown
    
    def update(self, delta_time):
        input_mgr = voidray.get_engine().input_manager
        
        # Get inputs
        throttle = 0
        if input_mgr.is_key_pressed(Keys.UP):
            throttle = 1
        if input_mgr.is_key_pressed(Keys.DOWN):
            throttle = -1
        
        steering = 0
        if input_mgr.is_key_pressed(Keys.LEFT):
            steering = -1
        if input_mgr.is_key_pressed(Keys.RIGHT):
            steering = 1
        
        if self.rigidbody:
            # Forward/backward movement
            if throttle != 0:
                # Get forward direction
                rotation_rad = math.radians(self.game_object.transform.rotation)
                forward = Vector2(math.cos(rotation_rad), math.sin(rotation_rad))
                
                # Apply force in forward direction
                force = forward * throttle * self.acceleration * self.rigidbody.mass
                self.rigidbody.add_force(force)
            
            # Steering (only when moving)
            current_speed = self.rigidbody.velocity.magnitude()
            if current_speed > 10 and steering != 0:  # Minimum speed for turning
                turn_amount = steering * self.turn_speed * delta_time
                # Scale turning by speed
                turn_amount *= min(current_speed / 100, 1.0)
                self.game_object.transform.rotate(turn_amount)
            
            # Simulate tire friction (drift prevention)
            velocity = self.rigidbody.velocity
            rotation_rad = math.radians(self.game_object.transform.rotation)
            forward = Vector2(math.cos(rotation_rad), math.sin(rotation_rad))
            right = Vector2(-forward.y, forward.x)
            
            # Reduce sideways velocity (drift control)
            sideways_velocity = velocity.dot(right)
            velocity -= right * (sideways_velocity * self.drift_factor * delta_time)
            
            self.rigidbody.set_velocity(velocity)
```

## 💥 Advanced Physics Techniques

### Explosion Physics
```python
def create_explosion(center_position, force_strength, radius):
    """Create an explosion that affects nearby objects"""
    # Get all objects in the explosion radius
    physics_system = voidray.get_engine().physics_system
    affected_objects = physics_system.get_rigidbodies_in_area(center_position, radius)
    
    for rigidbody in affected_objects:
        if rigidbody.game_object:
            # Calculate direction from explosion center
            direction = (rigidbody.game_object.transform.position - center_position)
            distance = direction.magnitude()
            
            if distance > 0:
                direction = direction.normalized()
                
                # Apply force with falloff based on distance
                falloff = 1.0 - (distance / radius)
                force = direction * force_strength * falloff * rigidbody.mass
                
                rigidbody.add_impulse(force)
```

### Joint System (Connect Objects)
```python
class SpringJoint(Component):
    """Connects two objects with a spring"""
    def __init__(self, target_object, spring_strength=100, max_distance=100):
        super().__init__()
        self.target = target_object
        self.spring_strength = spring_strength
        self.max_distance = max_distance
        self.rest_length = 50
    
    def start(self):
        self.rigidbody = self.game_object.get_component(Rigidbody)
    
    def update(self, delta_time):
        if not self.target or not self.rigidbody:
            return
        
        # Calculate spring force
        direction = (self.target.transform.position - 
                    self.game_object.transform.position)
        distance = direction.magnitude()
        
        if distance > 0:
            direction = direction.normalized()
            
            # Spring force based on distance from rest length
            force_magnitude = (distance - self.rest_length) * self.spring_strength
            spring_force = direction * force_magnitude
            
            self.rigidbody.add_force(spring_force)
            
            # Break joint if too far
            if distance > self.max_distance:
                self.game_object.remove_component(SpringJoint)
```

### One-Way Platforms
```python
class OneWayPlatform(Sprite):
    def __init__(self, width, height):
        super().__init__("OneWayPlatform")
        self.create_colored_rect(width, height, Color.GREEN)
        
        # Use trigger for custom collision handling
        self.collider = BoxCollider(width, height)
        self.collider.is_trigger = True
        self.collider.on_collision = self.check_platform_collision
        self.add_component(self.collider)
        
        self.platform_thickness = 10
    
    def check_platform_collision(self, other, collision_info):
        other_rigidbody = other.game_object.get_component(Rigidbody)
        if not other_rigidbody:
            return
        
        # Only collide if falling down onto platform
        if other_rigidbody.velocity.y > 0:  # Moving downward
            other_bottom = (other.game_object.transform.position.y + 
                          other.offset.y + other.height/2)
            platform_top = (self.transform.position.y - 
                          self.collider.height/2)
            
            # Check if object is close to platform top
            if abs(other_bottom - platform_top) < self.platform_thickness:
                # Stop the object on the platform
                velocity = other_rigidbody.velocity
                velocity.y = 0
                other_rigidbody.set_velocity(velocity)
                
                # Position object on platform
                other.game_object.transform.position.y = (platform_top - 
                                                         other.height/2 - other.offset.y)
```

## 🔧 Physics Performance Tips

### Optimization Strategies
```python
# 1. Use appropriate collider shapes
# Circles are faster than boxes for round objects
circle_collider = CircleCollider(radius)  # Faster
box_collider = BoxCollider(radius*2, radius*2)  # Slower for round objects

# 2. Put static objects to sleep
static_rigidbody.is_kinematic = True  # No physics simulation

# 3. Use object pooling for bullets/particles
class BulletPool:
    def __init__(self, size=50):
        self.bullets = []
        for _ in range(size):
            bullet = Bullet()
            bullet.active = False
            self.bullets.append(bullet)
    
    def get_bullet(self):
        for bullet in self.bullets:
            if not bullet.active:
                bullet.active = True
                return bullet
        return None  # Pool exhausted

# 4. Reduce physics timestep for better performance
physics_system.set_timestep(1.0 / 30.0)  # 30 FPS physics instead of 60
```

### Debugging Physics
```python
# Enable physics debug visualization
engine = voidray.get_engine()
engine.debug_overlay.show_colliders = True
engine.debug_overlay.show_velocities = True
engine.debug_overlay.show_forces = True

# Performance monitoring
def monitor_physics():
    stats = engine.physics_system.get_statistics()
    print(f"Active rigidbodies: {stats['active_rigidbodies']}")
    print(f"Sleeping rigidbodies: {stats['sleeping_rigidbodies']}")
    print(f"Total collisions: {stats.get('collision_count', 0)}")
```

## 📚 Next Steps

Master physics in VoidRay:

1. **[Animation System](animation.md)** - Combine physics with animations
2. **[Audio System](audio.md)** - Add sound to physics interactions
3. **[Particle Effects](particles.md)** - Visual effects for physics events
4. **[Platformer Guide](guide_platformer.md)** - Complete physics-based game

Physics makes your games feel alive and responsive. Experiment with different settings to find the perfect feel for your game! 🎮
