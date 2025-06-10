
# Your First Game with VoidRay

Welcome to VoidRay! This tutorial will guide you through creating your first complete game in just 15 minutes. No prior game development experience required!

## 🎯 What We're Building

A simple but fun **Catch the Coins** game where:
- You control a blue player with arrow keys
- Yellow coins fall from the sky
- Catch coins to increase your score
- Game gets faster as you play

## 🚀 Step 1: Basic Setup (2 minutes)

Create a new file called `my_first_game.py` and start with this foundation:

```python
import voidray
from voidray import Scene, GameObject, Vector2, Keys, BoxCollider
from voidray.graphics.sprite import Sprite
from voidray.utils.color import Color
import random

# We'll build our game step by step
def main():
    # Create a game window: width, height, title, fps
    voidray.configure(800, 600, "My First VoidRay Game", 60)
    voidray.on_init(start_game)
    voidray.start()

def start_game():
    # We'll add our game setup here
    pass

if __name__ == "__main__":
    main()
```

**Try it now!** Run this code - you should see an empty black window. That's your game canvas!

## 🎮 Step 2: Create Your Player (3 minutes)

Let's create a player character you can control:

```python
class Player(GameObject):
    def __init__(self):
        super().__init__("Player")
        # Create a blue square sprite
        self.sprite = Sprite()
        self.sprite.create_colored_rect(40, 40, Color.BLUE)
        self.add_component(self.sprite)
        
        self.speed = 300  # How fast the player moves
        self.score = 0    # Player's score
        
        # Add collision detection so we can catch coins
        self.collider = BoxCollider(40, 40)
        self.collider.on_collision = self.catch_coin
        self.add_component(self.collider)
    
    def update(self, delta_time):
        """This runs every frame to update the player"""
        super().update(delta_time)
        self.handle_movement(delta_time)
        self.stay_on_screen()
    
    def handle_movement(self, delta_time):
        """Move the player based on keyboard input"""
        input_mgr = voidray.get_engine().input_manager
        movement = Vector2.zero()
        
        # Check which keys are pressed
        if input_mgr.is_key_pressed(Keys.LEFT):
            movement.x = -1
        if input_mgr.is_key_pressed(Keys.RIGHT):
            movement.x = 1
        if input_mgr.is_key_pressed(Keys.UP):
            movement.y = -1
        if input_mgr.is_key_pressed(Keys.DOWN):
            movement.y = 1
        
        # Move the player
        velocity = movement * self.speed * delta_time
        self.transform.position += velocity
    
    def stay_on_screen(self):
        """Keep the player inside the game window"""
        engine = voidray.get_engine()
        pos = self.transform.position
        
        # Don't let player go off screen
        pos.x = max(20, min(engine.width - 20, pos.x))
        pos.y = max(20, min(engine.height - 20, pos.y))
    
    def catch_coin(self, other, collision_info):
        """What happens when player touches a coin"""
        if "Coin" in other.game_object.name:
            self.score += 10
            other.game_object.destroy()  # Remove the coin
            print(f"Score: {self.score}")
```

## 💰 Step 3: Create Falling Coins (3 minutes)

Now let's create coins that fall from the sky:

```python
class Coin(GameObject):
    def __init__(self):
        super().__init__("Coin")
        # Create a yellow circle sprite
        self.sprite = Sprite()
        self.sprite.create_colored_circle(12, Color.YELLOW)
        self.add_component(self.sprite)
        
        self.fall_speed = random.randint(100, 250)  # Random falling speed
        
        # Add collision detection
        self.collider = BoxCollider(24, 24)
        self.add_component(self.collider)
    
    def update(self, delta_time):
        """Make the coin fall down"""
        super().update(delta_time)
        
        # Move the coin downward
        self.transform.position.y += self.fall_speed * delta_time
        
        # Remove coin if it falls off screen
        engine = voidray.get_engine()
        if self.transform.position.y > engine.height + 50:
            self.destroy()

class CoinSpawner:
    """Creates new coins at random intervals"""
    def __init__(self, scene):
        self.scene = scene
        self.spawn_timer = 0
        self.spawn_interval = 1.5  # Seconds between coins
        self.game_speed = 1.0      # Game gets faster over time
    
    def update(self, delta_time):
        self.spawn_timer += delta_time
        
        # Time to spawn a new coin?
        if self.spawn_timer >= self.spawn_interval:
            self.spawn_coin()
            self.spawn_timer = 0
            
            # Make game gradually faster
            self.game_speed += 0.01
            self.spawn_interval = max(0.3, 1.5 / self.game_speed)
    
    def spawn_coin(self):
        """Create a new coin at the top of the screen"""
        coin = Coin()
        # Random X position at top of screen
        coin.transform.position = Vector2(
            random.randint(30, 770),  # Random X
            -30                       # Start above screen
        )
        # Increase fall speed based on game speed
        coin.fall_speed *= self.game_speed
        self.scene.add_object(coin)
```

## 🎬 Step 4: Create the Game Scene (4 minutes)

Now let's put everything together in a game scene:

```python
class GameScene(Scene):
    def __init__(self):
        super().__init__("Game")
        self.player = None
        self.coin_spawner = None
        self.game_time = 0
    
    def on_enter(self):
        """Set up the game when scene starts"""
        super().on_enter()
        
        # Create the player in the center bottom
        self.player = Player()
        self.player.transform.position = Vector2(400, 500)
        self.add_object(self.player)
        
        # Create the coin spawner
        self.coin_spawner = CoinSpawner(self)
        
        print("Game Started! Use arrow keys to catch falling coins!")
    
    def update(self, delta_time):
        """Update game every frame"""
        super().update(delta_time)
        self.game_time += delta_time
        
        # Update coin spawning
        if self.coin_spawner:
            self.coin_spawner.update(delta_time)
        
        # Check for quit
        input_mgr = voidray.get_engine().input_manager
        if input_mgr.is_key_just_pressed(Keys.ESCAPE):
            voidray.stop()
    
    def render(self, renderer):
        """Draw the game interface"""
        super().render(renderer)
        
        # Draw game UI
        renderer.draw_text("Catch the Coins!", Vector2(10, 10), Color.WHITE, 28)
        renderer.draw_text(f"Score: {self.player.score if self.player else 0}", 
                          Vector2(10, 50), Color.YELLOW, 24)
        renderer.draw_text(f"Time: {int(self.game_time)}s", 
                          Vector2(10, 80), Color.CYAN, 20)
        
        # Draw instructions
        renderer.draw_text("Arrow keys to move • ESC to quit", 
                          Vector2(10, 550), Color.LIGHT_GRAY, 16)

# Update the start_game function
def start_game():
    scene = GameScene()
    voidray.register_scene("game", scene)
    voidray.set_scene("game")
```

## 🎉 Step 5: Complete Game (3 minutes)

Here's your complete first game! Put it all together:

```python
import voidray
from voidray import Scene, GameObject, Vector2, Keys, BoxCollider
from voidray.graphics.sprite import Sprite
from voidray.utils.color import Color
import random

class Player(GameObject):
    def __init__(self):
        super().__init__("Player")
        self.sprite = Sprite()
        self.sprite.create_colored_rect(40, 40, Color.BLUE)
        self.add_component(self.sprite)
        
        self.speed = 300
        self.score = 0
        
        self.collider = BoxCollider(40, 40)
        self.collider.on_collision = self.catch_coin
        self.add_component(self.collider)
    
    def update(self, delta_time):
        super().update(delta_time)
        self.handle_movement(delta_time)
        self.stay_on_screen()
    
    def handle_movement(self, delta_time):
        input_mgr = voidray.get_engine().input_manager
        movement = Vector2.zero()
        
        if input_mgr.is_key_pressed(Keys.LEFT):
            movement.x = -1
        if input_mgr.is_key_pressed(Keys.RIGHT):
            movement.x = 1
        if input_mgr.is_key_pressed(Keys.UP):
            movement.y = -1
        if input_mgr.is_key_pressed(Keys.DOWN):
            movement.y = 1
        
        velocity = movement * self.speed * delta_time
        self.transform.position += velocity
    
    def stay_on_screen(self):
        engine = voidray.get_engine()
        pos = self.transform.position
        pos.x = max(20, min(engine.width - 20, pos.x))
        pos.y = max(20, min(engine.height - 20, pos.y))
    
    def catch_coin(self, other, collision_info):
        if "Coin" in other.game_object.name:
            self.score += 10
            other.game_object.destroy()
            print(f"Score: {self.score}")

class Coin(GameObject):
    def __init__(self):
        super().__init__("Coin")
        self.sprite = Sprite()
        self.sprite.create_colored_circle(12, Color.YELLOW)
        self.add_component(self.sprite)
        
        self.fall_speed = random.randint(100, 250)
        
        self.collider = BoxCollider(24, 24)
        self.add_component(self.collider)
    
    def update(self, delta_time):
        super().update(delta_time)
        self.transform.position.y += self.fall_speed * delta_time
        
        engine = voidray.get_engine()
        if self.transform.position.y > engine.height + 50:
            self.destroy()

class CoinSpawner:
    def __init__(self, scene):
        self.scene = scene
        self.spawn_timer = 0
        self.spawn_interval = 1.5
        self.game_speed = 1.0
    
    def update(self, delta_time):
        self.spawn_timer += delta_time
        
        if self.spawn_timer >= self.spawn_interval:
            self.spawn_coin()
            self.spawn_timer = 0
            
            self.game_speed += 0.01
            self.spawn_interval = max(0.3, 1.5 / self.game_speed)
    
    def spawn_coin(self):
        coin = Coin()
        coin.transform.position = Vector2(random.randint(30, 770), -30)
        coin.fall_speed *= self.game_speed
        self.scene.add_object(coin)

class GameScene(Scene):
    def __init__(self):
        super().__init__("Game")
        self.player = None
        self.coin_spawner = None
        self.game_time = 0
    
    def on_enter(self):
        super().on_enter()
        
        self.player = Player()
        self.player.transform.position = Vector2(400, 500)
        self.add_object(self.player)
        
        self.coin_spawner = CoinSpawner(self)
        
        print("Game Started! Use arrow keys to catch falling coins!")
    
    def update(self, delta_time):
        super().update(delta_time)
        self.game_time += delta_time
        
        if self.coin_spawner:
            self.coin_spawner.update(delta_time)
        
        input_mgr = voidray.get_engine().input_manager
        if input_mgr.is_key_just_pressed(Keys.ESCAPE):
            voidray.stop()
    
    def render(self, renderer):
        super().render(renderer)
        
        renderer.draw_text("Catch the Coins!", Vector2(10, 10), Color.WHITE, 28)
        renderer.draw_text(f"Score: {self.player.score if self.player else 0}", 
                          Vector2(10, 50), Color.YELLOW, 24)
        renderer.draw_text(f"Time: {int(self.game_time)}s", 
                          Vector2(10, 80), Color.CYAN, 20)
        renderer.draw_text("Arrow keys to move • ESC to quit", 
                          Vector2(10, 550), Color.LIGHT_GRAY, 16)

def start_game():
    scene = GameScene()
    voidray.register_scene("game", scene)
    voidray.set_scene("game")

def main():
    voidray.configure(800, 600, "My First VoidRay Game", 60)
    voidray.on_init(start_game)
    voidray.start()

if __name__ == "__main__":
    main()
```

## 🎮 Play Your Game!

Run your game with:
```bash
python my_first_game.py
```

**Controls:**
- Arrow keys to move
- ESC to quit

## 🎯 What You've Learned

Congratulations! You've just created a complete game that demonstrates:

1. **Game Objects**: Player and Coins with different behaviors
2. **Input Handling**: Responsive keyboard controls
3. **Collision Detection**: Player catching coins
4. **Game Logic**: Scoring, spawning, difficulty progression
5. **UI Rendering**: Score display and instructions

## 🚀 Next Steps

Ready to level up? Try these challenges:

### Easy Modifications:
- Change player color or size
- Make coins worth different points
- Add a high score display

### Medium Challenges:
- Add sound effects when catching coins
- Create power-ups that make player faster
- Add a game over condition (time limit or miss too many coins)

### Advanced Features:
- Multiple levels with different backgrounds
- Enemy objects to avoid
- Particle effects when catching coins

## 📚 Continue Learning

Your next stops in the VoidRay documentation:

1. **[Game Objects & Components](game_objects.md)** - Learn about the component system
2. **[Physics System](physics.md)** - Add realistic physics to your games
3. **[Input Handling](input.md)** - Master all types of input
4. **[Complete Game Tutorial](tutorial_basic_game.txt)** - Build a more complex game

You're now a VoidRay game developer! 🎉
