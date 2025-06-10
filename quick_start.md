# Quick Start Guide

Get up and running with VoidRay in just 10 minutes! This guide will walk you through creating your first interactive game.

## 🎯 What We'll Build

A simple but complete game featuring:
- A player character you can move with arrow keys
- Collectible items that disappear when touched
- A score system
- Basic physics and collision detection

## 🚀 Step 1: Basic Setup (2 minutes)

Create a new file called `my_first_game.py`:

```python
import voidray
from voidray import Scene, Sprite, Vector2, Keys, BoxCollider
from voidray.graphics.renderer import Color

# We'll add our game classes here

def main():
    voidray.configure(800, 600, "My First VoidRay Game", 60)
    voidray.on_init(init_game)
    voidray.start()

if __name__ == "__main__":
    main()
```

## 🎮 Step 2: Create the Player (3 minutes)

Add the player class:

```python
class Player(Sprite):
    def __init__(self):
        super().__init__("Player")
        # Create a blue square
        self.create_colored_rect(40, 40, Color.BLUE)
        self.speed = 200
        self.score = 0

        # Add collision detection
        self.collider = BoxCollider(40, 40)
        self.collider.on_collision = self.on_collision
        self.add_component(self.collider)

    def update(self, delta_time):
        super().update(delta_time)

        # Get input
        input_manager = voidray.get_engine().input_manager
        movement = Vector2.zero()

        # Arrow key movement
        if input_manager.is_key_pressed(Keys.LEFT):
            movement.x = -self.speed
        if input_manager.is_key_pressed(Keys.RIGHT):
            movement.x = self.speed
        if input_manager.is_key_pressed(Keys.UP):
            movement.y = -self.speed
        if input_manager.is_key_pressed(Keys.DOWN):
            movement.y = self.speed

        # Apply movement
        self.transform.position += movement * delta_time

        # Keep player on screen
        engine = voidray.get_engine()
        pos = self.transform.position
        pos.x = max(20, min(engine.width - 20, pos.x))
        pos.y = max(20, min(engine.height - 20, pos.y))

    def on_collision(self, other, collision_info):
        if "Coin" in other.game_object.name:
            # Collect the coin
            self.score += 10
            other.game_object.destroy()
            print(f"Score: {self.score}")
```

## 💰 Step 3: Create Collectible Coins (2 minutes)

Add the coin class:

```python
class Coin(Sprite):
    def __init__(self):
        super().__init__("Coin")
        # Create a yellow circle
        self.create_colored_circle(12, Color.YELLOW)

        # Add collision detection
        self.collider = BoxCollider(24, 24)
        self.add_component(self.collider)

        # Animation variables
        self.time = 0
        self.start_y = 0

    def update(self, delta_time):
        super().update(delta_time)

        # Bobbing animation
        self.time += delta_time
        if self.start_y == 0:
            self.start_y = self.transform.position.y

        bob_offset = math.sin(self.time * 3) * 5
        self.transform.position.y = self.start_y + bob_offset
```

## 🎬 Step 4: Create the Game Scene (2 minutes)

Add the scene and setup:

```python
import random
import math

class GameScene(Scene):
    def __init__(self):
        super().__init__("Game")
        self.player = None

    def on_enter(self):
        super().on_enter()

        # Create player
        self.player = Player()
        self.player.transform.position = Vector2(400, 300)
        self.add_object(self.player)

        # Create coins
        for i in range(8):
            coin = Coin()
            coin.transform.position = Vector2(
                random.randint(50, 750),
                random.randint(50, 550)
            )
            self.add_object(coin)

        print("Game started! Use arrow keys to move and collect coins!")

    def update(self, delta_time):
        super().update(delta_time)

        # Check for quit
        if voidray.get_engine().input_manager.is_key_just_pressed(Keys.ESCAPE):
            voidray.stop()

        # Check win condition
        coins_left = len([obj for obj in self.game_objects if "Coin" in obj.name])
        if coins_left == 0:
            print(f"You Win! Final Score: {self.player.score}")

    def render(self, renderer):
        super().render(renderer)

        # Draw UI
        renderer.draw_text("Collect All Coins!", Vector2(10, 10), Color.WHITE, 24)
        renderer.draw_text(f"Score: {self.player.score if self.player else 0}", 
                          Vector2(10, 40), Color.YELLOW, 20)

        # Instructions
        renderer.draw_text("Arrow keys to move, ESC to quit", 
                          Vector2(10, 570), Color.LIGHT_GRAY, 16)

def init_game():
    scene = GameScene()
    voidray.register_scene("game", scene)
    voidray.set_scene("game")
```

## 🎉 Step 5: Run Your Game! (1 minute)

Your complete game should look like this:

```python
import voidray
from voidray import Scene, Sprite, Vector2, Keys, BoxCollider
from voidray.graphics.renderer import Color
import random
import math

class Player(Sprite):
    def __init__(self):
        super().__init__("Player")
        # Create a blue square
        self.create_colored_rect(40, 40, Color.BLUE)
        self.speed = 200
        self.score = 0

        # Add collision detection
        self.collider = BoxCollider(40, 40)
        self.collider.on_collision = self.on_collision
        self.add_component(self.collider)

    def update(self, delta_time):
        super().update(delta_time)

        # Get input
        input_manager = voidray.get_engine().input_manager
        movement = Vector2.zero()

        # Arrow key movement
        if input_manager.is_key_pressed(Keys.LEFT):
            movement.x = -self.speed
        if input_manager.is_key_pressed(Keys.RIGHT):
            movement.x = self.speed
        if input_manager.is_key_pressed(Keys.UP):
            movement.y = -self.speed
        if input_manager.is_key_pressed(Keys.DOWN):
            movement.y = self.speed

        # Apply movement
        self.transform.position += movement * delta_time

        # Keep player on screen
        engine = voidray.get_engine()
        pos = self.transform.position
        pos.x = max(20, min(engine.width - 20, pos.x))
        pos.y = max(20, min(engine.height - 20, pos.y))

    def on_collision(self, other, collision_info):
        if "Coin" in other.game_object.name:
            # Collect the coin
            self.score += 10
            other.game_object.destroy()
            print(f"Score: {self.score}")

class Coin(Sprite):
    def __init__(self):
        super().__init__("Coin")
        # Create a yellow circle
        self.create_colored_circle(12, Color.YELLOW)

        # Add collision detection
        self.collider = BoxCollider(24, 24)
        self.add_component(self.collider)

        # Animation variables
        self.time = 0
        self.start_y = 0

    def update(self, delta_time):
        super().update(delta_time)

        # Bobbing animation
        self.time += delta_time
        if self.start_y == 0:
            self.start_y = self.transform.position.y

        bob_offset = math.sin(self.time * 3) * 5
        self.transform.position.y = self.start_y + bob_offset

class GameScene(Scene):
    def __init__(self):
        super().__init__("Game")
        self.player = None

    def on_enter(self):
        super().on_enter()

        # Create player
        self.player = Player()
        self.player.transform.position = Vector2(400, 300)
        self.add_object(self.player)

        # Create coins
        for i in range(8):
            coin = Coin()
            coin.transform.position = Vector2(
                random.randint(50, 750),
                random.randint(50, 550)
            )
            self.add_object(coin)

        print("Game started! Use arrow keys to move and collect coins!")

    def update(self, delta_time):
        super().update(delta_time)

        # Check for quit
        if voidray.get_engine().input_manager.is_key_just_pressed(Keys.ESCAPE):
            voidray.stop()

        # Check win condition
        coins_left = len([obj for obj in self.game_objects if "Coin" in obj.name])
        if coins_left == 0:
            print(f"You Win! Final Score: {self.player.score}")

    def render(self, renderer):
        super().render(renderer)

        # Draw UI
        renderer.draw_text("Collect All Coins!", Vector2(10, 10), Color.WHITE, 24)
        renderer.draw_text(f"Score: {self.player.score if self.player else 0}", 
                          Vector2(10, 40), Color.YELLOW, 20)

        # Instructions
        renderer.draw_text("Arrow keys to move, ESC to quit", 
                          Vector2(10, 570), Color.LIGHT_GRAY, 16)

def init_game():
    scene = GameScene()
    voidray.register_scene("game", scene)
    voidray.set_scene("game")

def main():
    voidray.configure(800, 600, "My First VoidRay Game", 60)
    voidray.on_init(init_game)
    voidray.start()

if __name__ == "__main__":
    main()
```

**Run it:**
```bash
python my_first_game.py