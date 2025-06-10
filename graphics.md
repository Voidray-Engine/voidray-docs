
# Graphics & Rendering

Learn how to create beautiful visuals in VoidRay! This guide covers everything from basic sprites to advanced rendering techniques.

## 🎨 Visual Overview

VoidRay's graphics system provides:
- **2D/2.5D Rendering**: Sprites with depth and layering
- **Flexible Camera System**: Following, zooming, and effects
- **Efficient Rendering**: Batch processing and culling
- **Visual Effects**: Particles, lighting, and post-processing

## 🖼️ Working with Sprites

### Creating Basic Sprites
```python
from voidray import Sprite
from voidray.graphics.renderer import Color

# Method 1: Colored shapes
player = Sprite("Player")
player.create_colored_rect(40, 40, Color.BLUE)
player.create_colored_circle(20, Color.RED)

# Method 2: Load from image file
enemy = Sprite("Enemy")
enemy.load_texture("assets/enemy.png")

# Method 3: Create from existing texture
texture = asset_loader.load_texture("player.png")
player = Sprite("Player")
player.texture = texture
```

### Sprite Properties
```python
# Basic properties
sprite.color = Color.RED           # Tint color
sprite.alpha = 0.8                # Transparency (0.0 to 1.0)
sprite.flip_x = True              # Mirror horizontally
sprite.flip_y = False             # Mirror vertically
sprite.layer = 10                 # Rendering order (higher = front)

# Transform properties
sprite.transform.position = Vector2(100, 200)
sprite.transform.rotation = 45    # Degrees
sprite.transform.scale = Vector2(2.0, 1.5)  # Width, height multipliers
```

### Advanced Sprite Techniques
```python
class AnimatedSprite(Sprite):
    def __init__(self, name):
        super().__init__(name)
        self.animation_frames = []
        self.current_frame = 0
        self.animation_speed = 10  # FPS
        self.animation_time = 0
        
        # Load animation frames
        for i in range(8):
            texture = asset_loader.load_texture(f"walk_frame_{i}.png")
            self.animation_frames.append(texture)
    
    def update(self, delta_time):
        super().update(delta_time)
        self.animate(delta_time)
    
    def animate(self, delta_time):
        if not self.animation_frames:
            return
        
        self.animation_time += delta_time
        frame_duration = 1.0 / self.animation_speed
        
        if self.animation_time >= frame_duration:
            self.current_frame = (self.current_frame + 1) % len(self.animation_frames)
            self.texture = self.animation_frames[self.current_frame]
            self.animation_time = 0

class BlinkingSprite(Sprite):
    def __init__(self, name):
        super().__init__(name)
        self.blink_speed = 2.0
        self.time = 0
    
    def update(self, delta_time):
        super().update(delta_time)
        self.time += delta_time
        
        # Oscillate alpha for blinking effect
        self.alpha = (math.sin(self.time * self.blink_speed) + 1) / 2
```

## 📷 Camera System

### Basic Camera Control
```python
# Get the main camera
camera = voidray.get_engine().camera

# Position and movement
camera.transform.position = Vector2(400, 300)
camera.transform.translate(Vector2(10, 0))

# Zoom
camera.set_zoom(2.0)  # 2x zoom
camera.zoom *= 1.1    # Zoom in by 10%

# Rotation
camera.transform.rotation = 15  # Degrees
```

### Following Camera
```python
class FollowCamera(Component):
    def __init__(self, target, smoothing=5.0, offset=Vector2.zero()):
        super().__init__()
        self.target = target
        self.smoothing = smoothing
        self.offset = offset
        self.camera = None
    
    def start(self):
        self.camera = voidray.get_engine().camera
    
    def update(self, delta_time):
        if not self.target or not self.camera:
            return
        
        # Calculate target position
        target_pos = self.target.transform.position + self.offset
        current_pos = self.camera.transform.position
        
        # Smooth interpolation
        if self.smoothing > 0:
            new_pos = current_pos.lerp(target_pos, self.smoothing * delta_time)
        else:
            new_pos = target_pos
        
        self.camera.transform.position = new_pos

# Usage
player = Sprite("Player")
follow_camera = FollowCamera(player, smoothing=3.0, offset=Vector2(0, -50))
# Add to a game object or scene
```

### Camera Effects
```python
class CameraShake(Component):
    def __init__(self, duration=0.5, intensity=10):
        super().__init__()
        self.duration = duration
        self.intensity = intensity
        self.time_remaining = 0
        self.original_position = Vector2.zero()
        self.camera = None
    
    def start(self):
        self.camera = voidray.get_engine().camera
        self.original_position = self.camera.transform.position
    
    def shake(self, duration=None, intensity=None):
        """Start a camera shake effect"""
        if duration:
            self.duration = duration
        if intensity:
            self.intensity = intensity
        
        self.time_remaining = self.duration
        self.original_position = self.camera.transform.position
    
    def update(self, delta_time):
        if self.time_remaining <= 0 or not self.camera:
            return
        
        self.time_remaining -= delta_time
        
        # Calculate shake offset
        shake_strength = (self.time_remaining / self.duration) * self.intensity
        offset = Vector2(
            random.uniform(-shake_strength, shake_strength),
            random.uniform(-shake_strength, shake_strength)
        )
        
        self.camera.transform.position = self.original_position + offset
        
        # Reset when shake is done
        if self.time_remaining <= 0:
            self.camera.transform.position = self.original_position

# Usage - trigger shake on events
class Player(Sprite):
    def take_damage(self):
        camera_shake = scene.get_component(CameraShake)
        if camera_shake:
            camera_shake.shake(duration=0.3, intensity=15)
```

### Camera Bounds
```python
class BoundedCamera(Component):
    def __init__(self, min_bounds, max_bounds):
        super().__init__()
        self.min_bounds = min_bounds  # Vector2
        self.max_bounds = max_bounds  # Vector2
        self.camera = None
    
    def start(self):
        self.camera = voidray.get_engine().camera
    
    def update(self, delta_time):
        if not self.camera:
            return
        
        # Clamp camera position to bounds
        pos = self.camera.transform.position
        pos.x = max(self.min_bounds.x, min(self.max_bounds.x, pos.x))
        pos.y = max(self.min_bounds.y, min(self.max_bounds.y, pos.y))
        self.camera.transform.position = pos
```

## 🎨 Colors and Materials

### Working with Colors
```python
from voidray.graphics.renderer import Color

# Predefined colors
sprite.color = Color.RED
sprite.color = Color.BLUE
sprite.color = Color.WHITE
sprite.color = Color.BLACK
sprite.color = Color.TRANSPARENT

# Custom colors
sprite.color = Color(255, 128, 0)        # Orange (RGB)
sprite.color = Color(255, 128, 0, 128)   # Semi-transparent orange (RGBA)

# Color manipulation
red_component = sprite.color.r
green_component = sprite.color.g
blue_component = sprite.color.b
alpha_component = sprite.color.a

# Color interpolation
start_color = Color.RED
end_color = Color.BLUE
t = 0.5  # 50% between colors
mixed_color = start_color.lerp(end_color, t)
```

### Dynamic Color Effects
```python
class RainbowEffect(Component):
    def __init__(self, speed=2.0):
        super().__init__()
        self.speed = speed
        self.time = 0
    
    def update(self, delta_time):
        self.time += delta_time
        
        # Create rainbow effect using HSV color space
        hue = (self.time * self.speed) % 1.0
        sprite = self.game_object
        
        # Convert HSV to RGB (simplified)
        if hue < 1/6:
            sprite.color = Color(255, int(hue * 6 * 255), 0)
        elif hue < 2/6:
            sprite.color = Color(int((2/6 - hue) * 6 * 255), 255, 0)
        elif hue < 3/6:
            sprite.color = Color(0, 255, int((hue - 2/6) * 6 * 255))
        elif hue < 4/6:
            sprite.color = Color(0, int((4/6 - hue) * 6 * 255), 255)
        elif hue < 5/6:
            sprite.color = Color(int((hue - 4/6) * 6 * 255), 0, 255)
        else:
            sprite.color = Color(255, 0, int((1 - hue) * 6 * 255))

class FlashEffect(Component):
    def __init__(self, flash_color=Color.WHITE, duration=0.1):
        super().__init__()
        self.flash_color = flash_color
        self.duration = duration
        self.original_color = Color.WHITE
        self.flash_time = 0
        self.is_flashing = False
    
    def start(self):
        if hasattr(self.game_object, 'color'):
            self.original_color = self.game_object.color
    
    def flash(self, color=None, duration=None):
        """Trigger a flash effect"""
        if color:
            self.flash_color = color
        if duration:
            self.duration = duration
        
        self.is_flashing = True
        self.flash_time = self.duration
        self.game_object.color = self.flash_color
    
    def update(self, delta_time):
        if not self.is_flashing:
            return
        
        self.flash_time -= delta_time
        
        if self.flash_time <= 0:
            self.is_flashing = False
            self.game_object.color = self.original_color
```

## 🌟 Visual Effects

### Particle Systems
```python
class SimpleParticle(Sprite):
    def __init__(self, position, velocity, lifetime=2.0):
        super().__init__("Particle")
        self.create_colored_circle(3, Color.WHITE)
        self.transform.position = position
        self.velocity = velocity
        self.lifetime = lifetime
        self.max_lifetime = lifetime
    
    def update(self, delta_time):
        super().update(delta_time)
        
        # Move particle
        self.transform.translate(self.velocity * delta_time)
        
        # Age particle
        self.lifetime -= delta_time
        
        # Fade out as it ages
        fade = self.lifetime / self.max_lifetime
        self.alpha = fade
        
        # Remove when expired
        if self.lifetime <= 0:
            self.destroy()

class ParticleEmitter(Component):
    def __init__(self, particle_count=20, spread=360, speed_range=(50, 150)):
        super().__init__()
        self.particle_count = particle_count
        self.spread = spread  # Degrees
        self.speed_range = speed_range
    
    def emit(self, position):
        """Emit particles from a position"""
        scene = self.game_object.scene
        if not scene:
            return
        
        for _ in range(self.particle_count):
            # Random direction
            angle = random.uniform(0, math.radians(self.spread))
            speed = random.uniform(self.speed_range[0], self.speed_range[1])
            
            velocity = Vector2(
                math.cos(angle) * speed,
                math.sin(angle) * speed
            )
            
            # Create particle
            particle = SimpleParticle(
                position=position,
                velocity=velocity,
                lifetime=random.uniform(1.0, 3.0)
            )
            
            scene.add_object(particle)

# Usage
class Explosion(Sprite):
    def __init__(self, position):
        super().__init__("Explosion")
        self.transform.position = position
        
        # Add particle emitter
        emitter = ParticleEmitter(particle_count=30, spread=360)
        self.add_component(emitter)
        
        # Emit particles immediately
        emitter.emit(position)
        
        # Remove explosion object after a delay
        self.lifetime = 3.0
    
    def update(self, delta_time):
        super().update(delta_time)
        self.lifetime -= delta_time
        if self.lifetime <= 0:
            self.destroy()
```

### Screen Effects
```python
class ScreenFade(Component):
    def __init__(self):
        super().__init__()
        self.fade_alpha = 0.0
        self.fade_speed = 1.0
        self.fade_color = Color.BLACK
        self.is_fading = False
        self.fade_in = False
    
    def fade_to_black(self, duration=1.0):
        """Fade screen to black"""
        self.fade_speed = 1.0 / duration
        self.is_fading = True
        self.fade_in = False
    
    def fade_from_black(self, duration=1.0):
        """Fade screen from black"""
        self.fade_speed = 1.0 / duration
        self.is_fading = True
        self.fade_in = True
        self.fade_alpha = 1.0
    
    def update(self, delta_time):
        if not self.is_fading:
            return
        
        if self.fade_in:
            self.fade_alpha -= self.fade_speed * delta_time
            if self.fade_alpha <= 0:
                self.fade_alpha = 0
                self.is_fading = False
        else:
            self.fade_alpha += self.fade_speed * delta_time
            if self.fade_alpha >= 1:
                self.fade_alpha = 1
                self.is_fading = False
    
    def render(self, renderer):
        if self.fade_alpha > 0:
            # Draw fade overlay
            engine = voidray.get_engine()
            overlay_color = Color(
                self.fade_color.r,
                self.fade_color.g,
                self.fade_color.b,
                int(255 * self.fade_alpha)
            )
            renderer.draw_rect(
                Vector2.zero(),
                Vector2(engine.width, engine.height),
                overlay_color,
                filled=True
            )
```

### Trail Effects
```python
class TrailRenderer(Component):
    def __init__(self, trail_length=10, trail_color=Color.WHITE):
        super().__init__()
        self.trail_length = trail_length
        self.trail_color = trail_color
        self.trail_points = []
        self.update_interval = 0.05  # Seconds between trail points
        self.time_since_update = 0
    
    def update(self, delta_time):
        self.time_since_update += delta_time
        
        if self.time_since_update >= self.update_interval:
            # Add current position to trail
            current_pos = self.game_object.transform.position.copy()
            self.trail_points.append(current_pos)
            
            # Limit trail length
            if len(self.trail_points) > self.trail_length:
                self.trail_points.pop(0)
            
            self.time_since_update = 0
    
    def render(self, renderer):
        if len(self.trail_points) < 2:
            return
        
        # Draw trail lines with fading alpha
        for i in range(1, len(self.trail_points)):
            alpha = i / len(self.trail_points)  # Fade towards head
            color = Color(
                self.trail_color.r,
                self.trail_color.g,
                self.trail_color.b,
                int(255 * alpha)
            )
            
            start_pos = self.trail_points[i-1]
            end_pos = self.trail_points[i]
            renderer.draw_line(start_pos, end_pos, color, width=2)
```

## 🎭 2.5D Rendering

### Depth and Layering
```python
# Layer-based rendering (simple)
background.layer = -10   # Far back
ground.layer = 0         # Default layer
player.layer = 10        # In front
ui_elements.layer = 100  # Front-most

# Advanced depth sorting
class DepthSortedSprite(Sprite):
    def __init__(self, name):
        super().__init__(name)
        self.depth = 0  # Z-coordinate for 2.5D
    
    def update(self, delta_time):
        super().update(delta_time)
        
        # Update layer based on Y position for isometric effect
        self.layer = int(self.transform.position.y)
```

### Isometric Projection
```python
class IsometricSprite(Sprite):
    def __init__(self, name):
        super().__init__(name)
        self.world_position = Vector2.zero()  # 2D world position
    
    def update_screen_position(self):
        """Convert world position to isometric screen position"""
        # Isometric projection formula
        iso_x = (self.world_position.x - self.world_position.y) * 0.5
        iso_y = (self.world_position.x + self.world_position.y) * 0.25
        
        self.transform.position = Vector2(iso_x, iso_y)
        
        # Update depth layer
        self.layer = int(self.world_position.y)
    
    def set_world_position(self, world_pos):
        self.world_position = world_pos
        self.update_screen_position()
```

## 🔧 Performance Optimization

### Sprite Batching
```python
class SpriteBatch:
    def __init__(self):
        self.sprites = []
        self.max_batch_size = 100
    
    def add_sprite(self, sprite):
        self.sprites.append(sprite)
        
        # Process batch when full
        if len(self.sprites) >= self.max_batch_size:
            self.flush()
    
    def flush(self):
        """Render all sprites in batch"""
        if not self.sprites:
            return
        
        # Sort by texture for efficient rendering
        self.sprites.sort(key=lambda s: s.texture_id if hasattr(s, 'texture_id') else 0)
        
        # Render all sprites
        for sprite in self.sprites:
            sprite.render()
        
        self.sprites.clear()
```

### Frustum Culling
```python
class ViewportCuller(Component):
    def __init__(self, margin=50):
        super().__init__()
        self.margin = margin
        self.camera = None
    
    def start(self):
        self.camera = voidray.get_engine().camera
    
    def is_visible(self, sprite):
        """Check if sprite is visible in viewport"""
        if not self.camera:
            return True
        
        # Get camera viewport bounds
        engine = voidray.get_engine()
        cam_pos = self.camera.transform.position
        zoom = self.camera.zoom
        
        viewport_width = engine.width / zoom
        viewport_height = engine.height / zoom
        
        viewport_left = cam_pos.x - viewport_width / 2 - self.margin
        viewport_right = cam_pos.x + viewport_width / 2 + self.margin
        viewport_top = cam_pos.y - viewport_height / 2 - self.margin
        viewport_bottom = cam_pos.y + viewport_height / 2 + self.margin
        
        # Check if sprite is in viewport
        sprite_pos = sprite.transform.position
        
        return (viewport_left <= sprite_pos.x <= viewport_right and
                viewport_top <= sprite_pos.y <= viewport_bottom)

# Usage in scene
class OptimizedScene(Scene):
    def __init__(self):
        super().__init__()
        self.culler = ViewportCuller()
    
    def render(self, renderer):
        # Only render visible sprites
        visible_sprites = []
        for obj in self.game_objects:
            if isinstance(obj, Sprite) and self.culler.is_visible(obj):
                visible_sprites.append(obj)
        
        # Render visible sprites
        for sprite in visible_sprites:
            sprite.render(renderer)
```

## 📚 Next Steps

Master VoidRay graphics:

1. **[Animation System](animation.md)** - Bring your graphics to life
2. **[Particle Effects](particles.md)** - Advanced visual effects
3. **[UI System](ui.md)** - Create beautiful interfaces
4. **[Performance Optimization](performance.md)** - Keep your graphics smooth

Graphics are what players see first - make them count! 🎨
