
# Examples & Demos

Learn VoidRay through practical examples! This collection shows you how to implement common game mechanics and patterns.

## 🎯 Getting Started Examples

### Hello World
The absolute minimum VoidRay program:

```python
import voidray
from voidray import Scene, Sprite, Vector2
from voidray.graphics.renderer import Color

class HelloScene(Scene):
    def on_enter(self):
        # Create a simple sprite
        hello_sprite = Sprite("Hello")
        hello_sprite.create_colored_rect(100, 50, Color.BLUE)
        hello_sprite.transform.position = Vector2(400, 300)
        self.add_object(hello_sprite)
    
    def render(self, renderer):
        super().render(renderer)
        renderer.draw_text("Hello VoidRay!", Vector2(350, 200), Color.WHITE, 24)

def init_game():
    scene = HelloScene()
    voidray.register_scene("hello", scene)
    voidray.set_scene("hello")

def main():
    voidray.configure(800, 600, "Hello VoidRay", 60)
    voidray.on_init(init_game)
    voidray.start()

if __name__ == "__main__":
    main()
```

### Moving Square
Basic movement with keyboard input:

```python
import voidray
from voidray import Scene, Sprite, Vector2, Keys, Component
from voidray.graphics.renderer import Color

class MoveController(Component):
    def __init__(self):
        super().__init__()
        self.speed = 200
    
    def update(self, delta_time):
        input_mgr = voidray.get_engine().input_manager
        movement = Vector2.zero()
        
        if input_mgr.is_key_pressed(Keys.W):
            movement.y = -1
        if input_mgr.is_key_pressed(Keys.S):
            movement.y = 1
        if input_mgr.is_key_pressed(Keys.A):
            movement.x = -1
        if input_mgr.is_key_pressed(Keys.D):
            movement.x = 1
        
        velocity = movement.normalized() * self.speed * delta_time
        self.game_object.transform.translate(velocity)

class MovingSquareScene(Scene):
    def on_enter(self):
        square = Sprite("Square")
        square.create_colored_rect(50, 50, Color.GREEN)
        square.transform.position = Vector2(400, 300)
        square.add_component(MoveController())
        self.add_object(square)
    
    def render(self, renderer):
        super().render(renderer)
        renderer.draw_text("Use WASD to move", Vector2(10, 10), Color.WHITE, 20)

def init_game():
    scene = MovingSquareScene()
    voidray.register_scene("main", scene)
    voidray.set_scene("main")

def main():
    voidray.configure(800, 600, "Moving Square", 60)
    voidray.on_init(init_game)
    voidray.start()

if __name__ == "__main__":
    main()
```

## 🎮 Game Mechanic Examples

### Platformer Character Controller
Complete platformer movement with jumping:

```python
import voidray
from voidray import Scene, Sprite, Vector2, Keys, BoxCollider, Rigidbody, Component
from voidray.graphics.renderer import Color

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
        
        horizontal = 0
        if input_mgr.is_key_pressed(Keys.A) or input_mgr.is_key_pressed(Keys.LEFT):
            horizontal = -1
        if input_mgr.is_key_pressed(Keys.D) or input_mgr.is_key_pressed(Keys.RIGHT):
            horizontal = 1
        
        if self.rigidbody:
            # Apply horizontal force
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
                velocity = self.rigidbody.velocity
                velocity.y = -self.jump_force
                self.rigidbody.set_velocity(velocity)
            
            self.on_ground = False
            self.time_since_grounded = self.coyote_time + 1
    
    def on_collision(self, other, collision_info):
        # Check if landing on something
        if collision_info.normal.y < -0.7:  # Normal pointing mostly up
            self.on_ground = True

class Platform(Sprite):
    def __init__(self, width, height):
        super().__init__("Platform")
        self.create_colored_rect(width, height, Color.BROWN)
        
        collider = BoxCollider(width, height)
        self.add_component(collider)

class PlatformerScene(Scene):
    def on_enter(self):
        super().on_enter()
        
        # Create player
        player = Sprite("Player")
        player.create_colored_rect(30, 40, Color.BLUE)
        player.transform.position = Vector2(100, 300)
        
        # Add physics and collision
        rigidbody = Rigidbody()
        rigidbody.set_mass(1.0)
        player.add_component(rigidbody)
        
        collider = BoxCollider(30, 40)
        player.add_component(collider)
        
        # Add movement controller
        player.add_component(PlatformerController())
        
        self.add_object(player)
        
        # Create platforms
        ground = Platform(800, 50)
        ground.transform.position = Vector2(400, 575)
        self.add_object(ground)
        
        platform1 = Platform(200, 20)
        platform1.transform.position = Vector2(300, 450)
        self.add_object(platform1)
        
        platform2 = Platform(150, 20)
        platform2.transform.position = Vector2(600, 350)
        self.add_object(platform2)
    
    def update(self, delta_time):
        super().update(delta_time)
        
        # Enable gravity
        engine = voidray.get_engine()
        engine.physics_system.set_gravity(Vector2(0, 800))
        
        # Quit on ESC
        input_mgr = engine.input_manager
        if input_mgr.is_key_just_pressed(Keys.ESCAPE):
            voidray.stop()
    
    def render(self, renderer):
        super().render(renderer)
        renderer.draw_text("Platformer Demo", Vector2(10, 10), Color.WHITE, 24)
        renderer.draw_text("A/D or Arrow Keys: Move", Vector2(10, 40), Color.LIGHT_GRAY, 16)
        renderer.draw_text("Space: Jump", Vector2(10, 60), Color.LIGHT_GRAY, 16)
        renderer.draw_text("ESC: Quit", Vector2(10, 80), Color.LIGHT_GRAY, 16)

def init_game():
    scene = PlatformerScene()
    voidray.register_scene("platformer", scene)
    voidray.set_scene("platformer")

def main():
    voidray.configure(800, 600, "Platformer Demo", 60)
    voidray.on_init(init_game)
    voidray.start()

if __name__ == "__main__":
    main()
```

### Space Shooter
Classic space shooter with bullets and enemies:

```python
import voidray
from voidray import Scene, Sprite, Vector2, Keys, BoxCollider, Component
from voidray.graphics.renderer import Color
import random
import math

class Bullet(Sprite):
    def __init__(self, direction, speed=500):
        super().__init__("Bullet")
        self.create_colored_rect(4, 10, Color.YELLOW)
        self.direction = direction
        self.speed = speed
        
        collider = BoxCollider(4, 10)
        collider.on_collision = self.on_hit
        self.add_component(collider)
    
    def update(self, delta_time):
        super().update(delta_time)
        
        # Move bullet
        velocity = self.direction * self.speed * delta_time
        self.transform.translate(velocity)
        
        # Remove if off screen
        pos = self.transform.position
        if pos.y < -10 or pos.y > 610 or pos.x < -10 or pos.x > 810:
            self.destroy()
    
    def on_hit(self, other, collision_info):
        if "Enemy" in other.game_object.name:
            other.game_object.destroy()
            self.destroy()

class Enemy(Sprite):
    def __init__(self):
        super().__init__("Enemy")
        self.create_colored_rect(30, 30, Color.RED)
        self.speed = random.randint(50, 150)
        
        collider = BoxCollider(30, 30)
        collider.on_collision = self.on_hit
        self.add_component(collider)
    
    def update(self, delta_time):
        super().update(delta_time)
        
        # Move down
        self.transform.position.y += self.speed * delta_time
        
        # Remove if off screen
        if self.transform.position.y > 650:
            self.destroy()
    
    def on_hit(self, other, collision_info):
        if "Player" in other.game_object.name:
            # Handle player collision
            player_controller = other.game_object.get_component(PlayerController)
            if player_controller:
                player_controller.take_damage()

class PlayerController(Component):
    def __init__(self):
        super().__init__()
        self.speed = 400
        self.health = 3
        self.fire_rate = 0.2
        self.time_since_shot = 0
        self.score = 0
    
    def update(self, delta_time):
        self.handle_movement(delta_time)
        self.handle_shooting(delta_time)
        self.time_since_shot += delta_time
    
    def handle_movement(self, delta_time):
        input_mgr = voidray.get_engine().input_manager
        movement = Vector2.zero()
        
        if input_mgr.is_key_pressed(Keys.A) or input_mgr.is_key_pressed(Keys.LEFT):
            movement.x = -1
        if input_mgr.is_key_pressed(Keys.D) or input_mgr.is_key_pressed(Keys.RIGHT):
            movement.x = 1
        if input_mgr.is_key_pressed(Keys.W) or input_mgr.is_key_pressed(Keys.UP):
            movement.y = -1
        if input_mgr.is_key_pressed(Keys.S) or input_mgr.is_key_pressed(Keys.DOWN):
            movement.y = 1
        
        velocity = movement.normalized() * self.speed * delta_time
        self.game_object.transform.translate(velocity)
        
        # Keep on screen
        pos = self.game_object.transform.position
        pos.x = max(15, min(785, pos.x))
        pos.y = max(15, min(585, pos.y))
    
    def handle_shooting(self, delta_time):
        input_mgr = voidray.get_engine().input_manager
        
        if input_mgr.is_key_pressed(Keys.SPACE) and self.time_since_shot >= self.fire_rate:
            self.shoot()
            self.time_since_shot = 0
    
    def shoot(self):
        bullet = Bullet(Vector2(0, -1))  # Shoot upward
        bullet_pos = self.game_object.transform.position
        bullet_pos.y -= 25  # Offset to front of ship
        bullet.transform.position = bullet_pos
        
        scene = self.game_object.scene
        scene.add_object(bullet)
    
    def take_damage(self):
        self.health -= 1
        print(f"Health: {self.health}")
        if self.health <= 0:
            print("Game Over!")
            # Could transition to game over scene here

class EnemySpawner:
    def __init__(self, scene):
        self.scene = scene
        self.spawn_timer = 0
        self.spawn_interval = 1.0
    
    def update(self, delta_time):
        self.spawn_timer += delta_time
        
        if self.spawn_timer >= self.spawn_interval:
            self.spawn_enemy()
            self.spawn_timer = 0
            
            # Gradually increase spawn rate
            self.spawn_interval = max(0.3, self.spawn_interval * 0.99)
    
    def spawn_enemy(self):
        enemy = Enemy()
        enemy.transform.position = Vector2(random.randint(30, 770), -30)
        self.scene.add_object(enemy)

class SpaceShooterScene(Scene):
    def __init__(self):
        super().__init__("SpaceShooter")
        self.player = None
        self.enemy_spawner = None
        self.score = 0
    
    def on_enter(self):
        super().on_enter()
        
        # Create player
        self.player = Sprite("Player")
        self.player.create_colored_rect(30, 30, Color.BLUE)
        self.player.transform.position = Vector2(400, 500)
        
        collider = BoxCollider(30, 30)
        self.player.add_component(collider)
        
        player_controller = PlayerController()
        self.player.add_component(player_controller)
        
        self.add_object(self.player)
        
        # Create enemy spawner
        self.enemy_spawner = EnemySpawner(self)
    
    def update(self, delta_time):
        super().update(delta_time)
        
        if self.enemy_spawner:
            self.enemy_spawner.update(delta_time)
        
        # Quit on ESC
        input_mgr = voidray.get_engine().input_manager
        if input_mgr.is_key_just_pressed(Keys.ESCAPE):
            voidray.stop()
    
    def render(self, renderer):
        super().render(renderer)
        
        # Get player stats
        player_controller = self.player.get_component(PlayerController) if self.player else None
        health = player_controller.health if player_controller else 0
        score = player_controller.score if player_controller else 0
        
        # Draw UI
        renderer.draw_text("Space Shooter", Vector2(10, 10), Color.WHITE, 24)
        renderer.draw_text(f"Health: {health}", Vector2(10, 40), Color.RED, 20)
        renderer.draw_text(f"Score: {score}", Vector2(10, 65), Color.YELLOW, 20)
        
        # Instructions
        renderer.draw_text("WASD/Arrows: Move", Vector2(10, 520), Color.LIGHT_GRAY, 16)
        renderer.draw_text("Space: Shoot", Vector2(10, 540), Color.LIGHT_GRAY, 16)
        renderer.draw_text("ESC: Quit", Vector2(10, 560), Color.LIGHT_GRAY, 16)

def init_game():
    scene = SpaceShooterScene()
    voidray.register_scene("shooter", scene)
    voidray.set_scene("shooter")

def main():
    voidray.configure(800, 600, "Space Shooter", 60)
    voidray.on_init(init_game)
    voidray.start()

if __name__ == "__main__":
    main()
```

### Breakout/Pong Physics
Ball physics with paddle collision:

```python
import voidray
from voidray import Scene, Sprite, Vector2, Keys, BoxCollider, Component
from voidray.graphics.renderer import Color
import math

class Ball(Sprite):
    def __init__(self):
        super().__init__("Ball")
        self.create_colored_circle(10, Color.WHITE)
        self.velocity = Vector2(200, -300)  # Starting velocity
        self.max_speed = 500
        
        collider = BoxCollider(20, 20)
        collider.on_collision = self.on_collision
        self.add_component(collider)
    
    def update(self, delta_time):
        super().update(delta_time)
        
        # Move ball
        self.transform.translate(self.velocity * delta_time)
        
        # Bounce off screen edges
        pos = self.transform.position
        engine = voidray.get_engine()
        
        # Left/right walls
        if pos.x <= 10:
            pos.x = 10
            self.velocity.x = abs(self.velocity.x)
        elif pos.x >= engine.width - 10:
            pos.x = engine.width - 10
            self.velocity.x = -abs(self.velocity.x)
        
        # Top wall
        if pos.y <= 10:
            pos.y = 10
            self.velocity.y = abs(self.velocity.y)
        
        # Bottom wall (game over)
        if pos.y >= engine.height + 50:
            self.reset_ball()
    
    def on_collision(self, other, collision_info):
        if "Paddle" in other.game_object.name:
            self.bounce_off_paddle(other.game_object)
        elif "Block" in other.game_object.name:
            self.bounce_off_block(collision_info)
            other.game_object.destroy()
    
    def bounce_off_paddle(self, paddle):
        # Calculate bounce angle based on where ball hits paddle
        paddle_pos = paddle.transform.position
        ball_pos = self.transform.position
        
        # Distance from center of paddle (-1 to 1)
        hit_pos = (ball_pos.x - paddle_pos.x) / 50  # Assuming paddle width is 100
        hit_pos = max(-1, min(1, hit_pos))  # Clamp to range
        
        # Calculate new velocity
        speed = self.velocity.magnitude()
        angle = hit_pos * 60  # Max 60 degree angle
        
        self.velocity.x = math.sin(math.radians(angle)) * speed
        self.velocity.y = -abs(math.cos(math.radians(angle)) * speed)
        
        # Limit speed
        if self.velocity.magnitude() > self.max_speed:
            self.velocity = self.velocity.normalized() * self.max_speed
    
    def bounce_off_block(self, collision_info):
        # Simple bounce based on collision normal
        normal = collision_info.normal
        
        # Reflect velocity
        dot = self.velocity.dot(normal)
        self.velocity = self.velocity - (normal * (2 * dot))
    
    def reset_ball(self):
        engine = voidray.get_engine()
        self.transform.position = Vector2(engine.width // 2, engine.height // 2)
        self.velocity = Vector2(200, -300)

class Paddle(Sprite):
    def __init__(self):
        super().__init__("Paddle")
        self.create_colored_rect(100, 15, Color.BLUE)
        self.speed = 400
        
        collider = BoxCollider(100, 15)
        self.add_component(collider)
    
    def update(self, delta_time):
        super().update(delta_time)
        
        # Move with mouse
        input_mgr = voidray.get_engine().input_manager
        mouse_pos = input_mgr.get_mouse_position()
        
        # Follow mouse X position
        target_x = mouse_pos.x
        current_x = self.transform.position.x
        
        # Smooth movement
        diff = target_x - current_x
        movement = min(abs(diff), self.speed * delta_time)
        if diff < 0:
            movement = -movement
        
        self.transform.position.x += movement
        
        # Keep on screen
        engine = voidray.get_engine()
        pos = self.transform.position
        pos.x = max(50, min(engine.width - 50, pos.x))

class Block(Sprite):
    def __init__(self, color=Color.RED):
        super().__init__("Block")
        self.create_colored_rect(60, 20, color)
        
        collider = BoxCollider(60, 20)
        self.add_component(collider)

class BreakoutScene(Scene):
    def __init__(self):
        super().__init__("Breakout")
        self.ball = None
        self.paddle = None
        self.blocks_remaining = 0
    
    def on_enter(self):
        super().on_enter()
        
        # Create ball
        self.ball = Ball()
        self.ball.transform.position = Vector2(400, 300)
        self.add_object(self.ball)
        
        # Create paddle
        self.paddle = Paddle()
        self.paddle.transform.position = Vector2(400, 550)
        self.add_object(self.paddle)
        
        # Create blocks
        self.create_blocks()
    
    def create_blocks(self):
        colors = [Color.RED, Color.ORANGE, Color.YELLOW, Color.GREEN, Color.BLUE]
        
        for row in range(5):
            for col in range(12):
                block = Block(colors[row])
                x = 100 + col * 70
                y = 100 + row * 30
                block.transform.position = Vector2(x, y)
                self.add_object(block)
                self.blocks_remaining += 1
    
    def update(self, delta_time):
        super().update(delta_time)
        
        # Check win condition
        current_blocks = len([obj for obj in self.game_objects if "Block" in obj.name])
        if current_blocks == 0:
            print("You Win!")
            # Could restart or go to win screen
        
        # Quit on ESC
        input_mgr = voidray.get_engine().input_manager
        if input_mgr.is_key_just_pressed(Keys.ESCAPE):
            voidray.stop()
    
    def render(self, renderer):
        super().render(renderer)
        
        # Count remaining blocks
        blocks_left = len([obj for obj in self.game_objects if "Block" in obj.name])
        
        renderer.draw_text("Breakout", Vector2(10, 10), Color.WHITE, 24)
        renderer.draw_text(f"Blocks: {blocks_left}", Vector2(10, 40), Color.YELLOW, 20)
        renderer.draw_text("Move mouse to control paddle", Vector2(10, 570), Color.LIGHT_GRAY, 16)

def init_game():
    scene = BreakoutScene()
    voidray.register_scene("breakout", scene)
    voidray.set_scene("breakout")

def main():
    voidray.configure(800, 600, "Breakout", 60)
    voidray.on_init(init_game)
    voidray.start()

if __name__ == "__main__":
    main()
```

## 🎯 Quick Tips for Examples

### Running Examples
1. Copy any example code to a new `.py` file
2. Save it in your project folder
3. Run with `python filename.py`

### Modifying Examples
- Change colors by editing `Color.BLUE` to `Color.RED`, etc.
- Adjust speeds by changing values like `self.speed = 300`
- Modify sizes by changing `create_colored_rect(width, height, color)`
- Add new features by copying patterns from other examples

### Learning from Examples
- Read the comments to understand what each part does
- Try changing one thing at a time to see the effect
- Combine elements from different examples
- Use these as starting points for your own games

## 📚 Next Steps

After trying these examples:

1. **[First Game Tutorial](first_game_tutorial.md)** - Build a complete game step by step
2. **[Beginner Concepts](beginner_concepts.md)** - Understand the theory behind the code
3. **[Game Objects & Components](game_objects.md)** - Learn the component system in depth
4. **[Physics System](physics.md)** - Add realistic physics to your games

Ready to create your own games? Start with the simplest example and work your way up! 🚀
