
# Scene Management

Scenes are the organizational backbone of your VoidRay game. They represent different game states like menus, levels, cutscenes, or any distinct part of your game. This guide covers everything from basic scene creation to advanced scene management techniques.

## 🎬 Understanding Scenes

### What are Scenes?
A **Scene** is a container that holds:
- GameObjects (players, enemies, UI elements)
- Environment settings (lighting, physics)
- Game logic specific to that state
- Transition behaviors

### Common Scene Types
- **Game Scenes**: Actual gameplay levels
- **Menu Scenes**: Main menu, pause menu, settings
- **Cutscene Scenes**: Story sequences, dialogue
- **Loading Scenes**: Asset loading screens
- **UI Overlay Scenes**: HUD, inventory panels

## 🏗️ Creating Your First Scene

### Basic Scene Structure
```python
from voidray import Scene, Sprite, Vector2
from voidray.graphics.renderer import Color

class GameScene(Scene):
    def __init__(self):
        super().__init__("GameLevel1")
        self.player = None
        self.score = 0
        self.level_complete = False
    
    def on_enter(self):
        """Called when entering this scene"""
        super().on_enter()
        print(f"Entering {self.name}")
        
        # Create game objects
        self.setup_player()
        self.setup_enemies()
        self.setup_environment()
        
        # Initialize game state
        self.score = 0
        self.level_complete = False
    
    def on_exit(self):
        """Called when leaving this scene"""
        super().on_exit()
        print(f"Exiting {self.name}")
        
        # Cleanup resources
        self.save_progress()
        self.cleanup_audio()
    
    def update(self, delta_time):
        """Called every frame"""
        super().update(delta_time)
        
        # Scene-specific game logic
        self.check_win_condition()
        self.handle_input()
        self.update_ui()
    
    def render(self, renderer):
        """Called every frame for drawing"""
        super().render(renderer)
        
        # Custom rendering (UI, effects, etc.)
        self.draw_hud(renderer)
        self.draw_debug_info(renderer)
    
    def setup_player(self):
        """Create and configure the player"""
        self.player = Player()
        self.player.transform.position = Vector2(100, 300)
        self.add_object(self.player)
    
    def setup_enemies(self):
        """Create enemies for this level"""
        for i in range(5):
            enemy = Enemy()
            enemy.transform.position = Vector2(300 + i * 100, 300)
            self.add_object(enemy)
    
    def setup_environment(self):
        """Create level geometry and decorations"""
        # Ground platform
        ground = Platform(800, 50)
        ground.transform.position = Vector2(400, 550)
        self.add_object(ground)
        
        # Collectibles
        for i in range(8):
            coin = Collectible()
            coin.transform.position = Vector2(50 + i * 90, 200)
            self.add_object(coin)
```

### Registering and Using Scenes
```python
def init_game():
    # Create scenes
    menu_scene = MenuScene()
    game_scene = GameScene()
    game_over_scene = GameOverScene()
    
    # Register with the engine
    voidray.register_scene("menu", menu_scene)
    voidray.register_scene("game", game_scene)
    voidray.register_scene("game_over", game_over_scene)
    
    # Start with menu
    voidray.set_scene("menu")

def main():
    voidray.configure(800, 600, "My Game", 60)
    voidray.on_init(init_game)
    voidray.start()
```

## 🔄 Scene Transitions

### Basic Scene Switching
```python
# Simple scene change
voidray.set_scene("game")

# Scene switching from within a scene
class MenuScene(Scene):
    def update(self, delta_time):
        super().update(delta_time)
        
        input_mgr = voidray.get_engine().input_manager
        if input_mgr.is_key_just_pressed(Keys.ENTER):
            voidray.set_scene("game")
        elif input_mgr.is_key_just_pressed(Keys.ESCAPE):
            voidray.stop()
```

### Advanced Scene Transitions
```python
from voidray.core.scene_transitions import SceneTransition

class FadeTransition(SceneTransition):
    def __init__(self, duration=1.0, color=Color.BLACK):
        super().__init__(duration)
        self.color = color
    
    def render(self, renderer, progress):
        """Custom transition effect"""
        # Fade to black and back
        if progress < 0.5:
            alpha = int(255 * (progress * 2))
        else:
            alpha = int(255 * (1.0 - (progress - 0.5) * 2))
        
        overlay = Color(self.color.r, self.color.g, self.color.b, alpha)
        renderer.draw_rect(Vector2.zero(), 
                          Vector2(renderer.screen_width, renderer.screen_height),
                          overlay, filled=True)

# Use transition when changing scenes
transition = FadeTransition(duration=1.5, color=Color.BLACK)
scene_manager.load_scene("next_level", transition)
```

### Scene Stack (Pause/Resume)
```python
class PauseMenu(Scene):
    def __init__(self):
        super().__init__("PauseMenu")
        self.previous_scene = None
    
    def on_enter(self):
        super().on_enter()
        # Create pause menu UI
        self.create_pause_ui()
    
    def update(self, delta_time):
        super().update(delta_time)
        
        input_mgr = voidray.get_engine().input_manager
        if input_mgr.is_key_just_pressed(Keys.ESCAPE):
            # Return to previous scene
            self.resume_game()
    
    def resume_game(self):
        # Pop this scene and return to game
        scene_manager = voidray.get_engine().scene_manager
        scene_manager.pop_scene()

# In your game scene, push pause menu
class GameScene(Scene):
    def update(self, delta_time):
        super().update(delta_time)
        
        input_mgr = voidray.get_engine().input_manager
        if input_mgr.is_key_just_pressed(Keys.ESCAPE):
            # Push pause menu onto stack
            scene_manager = voidray.get_engine().scene_manager
            scene_manager.push_scene("pause")
```

## 🎮 Scene Types and Patterns

### Menu Scene
```python
class MainMenu(Scene):
    def __init__(self):
        super().__init__("MainMenu")
        self.buttons = []
        self.selected_button = 0
    
    def on_enter(self):
        super().on_enter()
        
        # Create menu background
        background = Sprite("Background")
        background.create_colored_rect(800, 600, Color.DARK_GRAY)
        background.transform.position = Vector2(400, 300)
        self.add_object(background)
        
        # Create title
        self.title_text = "MY AWESOME GAME"
        
        # Menu options
        self.menu_options = [
            ("Start Game", self.start_game),
            ("Settings", self.open_settings),
            ("Quit", self.quit_game)
        ]
        
        self.selected_option = 0
    
    def update(self, delta_time):
        super().update(delta_time)
        
        input_mgr = voidray.get_engine().input_manager
        
        # Navigation
        if input_mgr.is_key_just_pressed(Keys.UP):
            self.selected_option = (self.selected_option - 1) % len(self.menu_options)
        elif input_mgr.is_key_just_pressed(Keys.DOWN):
            self.selected_option = (self.selected_option + 1) % len(self.menu_options)
        
        # Selection
        if input_mgr.is_key_just_pressed(Keys.ENTER):
            option_name, callback = self.menu_options[self.selected_option]
            callback()
    
    def render(self, renderer):
        super().render(renderer)
        
        # Draw title
        renderer.draw_text(self.title_text, Vector2(400, 150), 
                          Color.WHITE, 48, center=True)
        
        # Draw menu options
        for i, (option_name, _) in enumerate(self.menu_options):
            y_pos = 300 + i * 60
            color = Color.YELLOW if i == self.selected_option else Color.WHITE
            renderer.draw_text(option_name, Vector2(400, y_pos), 
                              color, 24, center=True)
    
    def start_game(self):
        voidray.set_scene("game")
    
    def open_settings(self):
        voidray.set_scene("settings")
    
    def quit_game(self):
        voidray.stop()
```

### Loading Scene
```python
class LoadingScene(Scene):
    def __init__(self, target_scene):
        super().__init__("LoadingScene")
        self.target_scene = target_scene
        self.loading_progress = 0.0
        self.assets_to_load = []
        self.loaded_assets = 0
    
    def on_enter(self):
        super().on_enter()
        
        # Define assets to load for target scene
        if self.target_scene == "game":
            self.assets_to_load = [
                "assets/player.png",
                "assets/enemy.png",
                "assets/background.png",
                "assets/music.mp3",
                "assets/jump.wav"
            ]
        
        # Start loading assets
        self.start_loading()
    
    def start_loading(self):
        # Load assets asynchronously (simplified example)
        asset_loader = voidray.get_engine().asset_loader
        
        for asset_path in self.assets_to_load:
            try:
                asset_loader.load_async(asset_path, self.on_asset_loaded)
            except Exception as e:
                print(f"Failed to load {asset_path}: {e}")
                self.on_asset_loaded()
    
    def on_asset_loaded(self):
        self.loaded_assets += 1
        self.loading_progress = self.loaded_assets / len(self.assets_to_load)
        
        # Check if loading complete
        if self.loaded_assets >= len(self.assets_to_load):
            self.loading_complete()
    
    def loading_complete(self):
        # Switch to target scene
        voidray.set_scene(self.target_scene)
    
    def render(self, renderer):
        super().render(renderer)
        
        # Draw loading screen
        renderer.draw_text("Loading...", Vector2(400, 200), 
                          Color.WHITE, 32, center=True)
        
        # Progress bar
        bar_width = 400
        bar_height = 20
        bar_x = 200
        bar_y = 300
        
        # Background
        renderer.draw_rect(Vector2(bar_x, bar_y), Vector2(bar_width, bar_height),
                          Color.GRAY, filled=True)
        
        # Progress
        progress_width = bar_width * self.loading_progress
        renderer.draw_rect(Vector2(bar_x, bar_y), Vector2(progress_width, bar_height),
                          Color.GREEN, filled=True)
        
        # Percentage
        percentage = int(self.loading_progress * 100)
        renderer.draw_text(f"{percentage}%", Vector2(400, 350),
                          Color.WHITE, 18, center=True)
```

### Level Scene with Multiple Areas
```python
class LevelScene(Scene):
    def __init__(self, level_number):
        super().__init__(f"Level{level_number}")
        self.level_number = level_number
        self.current_area = 0
        self.areas = []
        self.player = None
    
    def on_enter(self):
        super().on_enter()
        
        # Load level data
        level_data = self.load_level_data()
        self.setup_areas(level_data)
        
        # Create persistent objects (player)
        self.player = Player()
        self.add_object(self.player)
        
        # Load first area
        self.load_area(0)
    
    def setup_areas(self, level_data):
        """Setup different areas within the level"""
        self.areas = [
            {
                'name': 'Forest',
                'enemies': 5,
                'background': 'forest_bg.png',
                'music': 'forest_music.mp3'
            },
            {
                'name': 'Cave',
                'enemies': 8,
                'background': 'cave_bg.png',
                'music': 'cave_music.mp3'
            },
            {
                'name': 'Boss Room',
                'enemies': 1,
                'background': 'boss_bg.png',
                'music': 'boss_music.mp3'
            }
        ]
    
    def load_area(self, area_index):
        if area_index >= len(self.areas):
            return
        
        # Clear previous area objects (except player)
        self.clear_area_objects()
        
        # Load new area
        area = self.areas[area_index]
        self.current_area = area_index
        
        # Change background
        self.setup_background(area['background'])
        
        # Create enemies
        self.create_enemies(area['enemies'])
        
        # Change music
        audio_mgr = voidray.get_engine().audio_manager
        audio_mgr.play_music(f"assets/{area['music']}")
        
        print(f"Loaded area: {area['name']}")
    
    def clear_area_objects(self):
        """Remove area-specific objects but keep persistent ones"""
        objects_to_remove = []
        for obj in self.game_objects:
            if obj.name not in ["Player", "UI"]:
                objects_to_remove.append(obj)
        
        for obj in objects_to_remove:
            self.remove_object(obj)
    
    def check_area_transition(self):
        """Check if player should move to next area"""
        if not self.player:
            return
        
        player_pos = self.player.transform.position
        
        # Simple area transition based on X position
        if player_pos.x > 750 and self.current_area < len(self.areas) - 1:
            self.load_area(self.current_area + 1)
            # Move player to start of new area
            self.player.transform.position.x = 50
        elif player_pos.x < 50 and self.current_area > 0:
            self.load_area(self.current_area - 1)
            # Move player to end of previous area
            self.player.transform.position.x = 750
```

## 🎯 Scene Communication

### Passing Data Between Scenes
```python
class SceneData:
    """Global data that persists between scenes"""
    def __init__(self):
        self.player_health = 100
        self.player_score = 0
        self.current_level = 1
        self.inventory = []
        self.game_settings = {}

# Create global game data
game_data = SceneData()

class GameScene(Scene):
    def on_enter(self):
        super().on_enter()
        
        # Use global data
        self.player.health = game_data.player_health
        self.score = game_data.player_score
    
    def on_exit(self):
        super().on_exit()
        
        # Save data for next scene
        game_data.player_health = self.player.health
        game_data.player_score = self.score

class GameOverScene(Scene):
    def on_enter(self):
        super().on_enter()
        
        # Access final score
        self.final_score = game_data.player_score
```

### Scene Events
```python
from voidray.core.event_system import EventSystem

class GameScene(Scene):
    def on_enter(self):
        super().on_enter()
        
        # Listen for events
        EventSystem.subscribe("player_died", self.handle_player_death)
        EventSystem.subscribe("level_complete", self.handle_level_complete)
    
    def handle_player_death(self, event_data):
        print("Player died! Transitioning to game over.")
        voidray.set_scene("game_over")
    
    def handle_level_complete(self, event_data):
        print("Level complete! Loading next level.")
        next_level = event_data.get("next_level", "level2")
        voidray.set_scene(next_level)

# Trigger events from anywhere in your game
class Player(Sprite):
    def take_damage(self, amount):
        self.health -= amount
        if self.health <= 0:
            EventSystem.emit("player_died", {"final_score": self.score})
```

## 🔧 Scene Performance and Best Practices

### Scene Optimization
```python
class OptimizedScene(Scene):
    def __init__(self):
        super().__init__("OptimizedScene")
        self.object_pools = {}
        self.cached_components = {}
    
    def on_enter(self):
        super().on_enter()
        
        # Pre-allocate object pools
        self.setup_object_pools()
        
        # Cache frequently accessed components
        self.cache_components()
    
    def setup_object_pools(self):
        """Pre-create objects for pooling"""
        # Bullet pool
        self.object_pools['bullets'] = []
        for i in range(50):
            bullet = Bullet()
            bullet.active = False
            self.object_pools['bullets'].append(bullet)
        
        # Particle pool
        self.object_pools['particles'] = []
        for i in range(100):
            particle = Particle()
            particle.active = False
            self.object_pools['particles'].append(particle)
    
    def get_pooled_object(self, object_type):
        """Get an object from the pool"""
        pool = self.object_pools.get(object_type, [])
        
        for obj in pool:
            if not obj.active:
                obj.active = True
                return obj
        
        return None  # Pool exhausted
    
    def return_to_pool(self, obj, object_type):
        """Return object to pool"""
        obj.active = False
        obj.reset()  # Custom reset method
```

### Memory Management
```python
class Scene(Scene):
    def on_exit(self):
        super().on_exit()
        
        # Explicit cleanup
        self.cleanup_resources()
    
    def cleanup_resources(self):
        """Clean up scene-specific resources"""
        # Clear large textures
        for obj in self.game_objects:
            if hasattr(obj, 'texture') and obj.texture:
                obj.texture = None
        
        # Stop audio
        audio_mgr = voidray.get_engine().audio_manager
        audio_mgr.stop_all_sounds()
        
        # Clear cached data
        self.cached_data = {}
        
        # Force garbage collection
        import gc
        gc.collect()
```

## 📚 Next Steps

Master scene management in VoidRay:

1. **[Input Handling](input.md)** - Handle input across different scenes
2. **[UI System](ui.md)** - Create scene-specific interfaces
3. **[Audio System](audio.md)** - Scene-based music and sound
4. **[Game Examples](examples.md)** - See complete scene implementations

Scenes are the foundation of game organization. Master them to create polished, professional games! 🎬
