
# Installation & Setup

This guide will help you install and set up VoidRay on your system. VoidRay works on Windows, macOS, and Linux.

## 🚀 Quick Setup on Replit

The easiest way to get started with VoidRay is on Replit:

1. **Fork this Repl** or create a new Python Repl
2. **Clone/Import** the VoidRay repository
3. **Click Run** - all dependencies install automatically
4. **Start coding** your game immediately!

### Advantages of Replit:
- ✅ No installation required
- ✅ Works on any device with a browser
- ✅ Automatic dependency management
- ✅ Built-in code editor
- ✅ One-click deployment
- ✅ Collaborative development

## 💻 Local Installation

### Prerequisites

- **Python 3.8+** (Python 3.12+ recommended)
- **pip** (Python package manager)

### Installation Steps

1. **Install VoidRay**:
```bash
pip install voidray
```

2. **Install dependencies** (if not auto-installed):
```bash
pip install pygame numpy
```

3. **Verify installation**:
```python
import voidray
print(f"VoidRay version: {voidray.__version__}")
```

### Manual Installation

If you're working with the source code:

1. **Clone the repository**:
```bash
git clone https://github.com/Voidray-Engine/Voidray.git
cd voidray
```

2. **Install in development mode**:
```bash
pip install -e .
```

## 🔧 Development Environment Setup

### Recommended Code Editors

1. **Built-in VoidRay Editor** (Recommended):
```bash
python gui_editor.py
```
Features:
- Syntax highlighting for VoidRay API
- Live error checking
- Project templates
- Asset browser
- Debug tools

2. **VS Code** with Python extension
3. **PyCharm** (Community or Professional)
4. **Sublime Text** with Python package

### Project Structure

Create a new project with this structure:
```
my_game/
├── assets/
│   ├── images/
│   ├── sounds/
│   └── fonts/
├── scripts/
├── saves/
├── main.py
└── config.json
```

## ✅ Verification

Test your installation with this simple script:

```python
import voidray
from voidray import Scene, Sprite, Vector2
from voidray.graphics.renderer import Color

class TestScene(Scene):
    def on_enter(self):
        super().on_enter()
        # Create a test sprite
        test_sprite = Sprite("test")
        test_sprite.create_colored_rect(50, 50, Color.BLUE)
        test_sprite.transform.position = Vector2(100, 100)
        self.add_object(test_sprite)

def init_game():
    scene = TestScene()
    voidray.register_scene("test", scene)
    voidray.set_scene("test")

def main():
    voidray.configure(800, 600, "VoidRay Test", 60)
    voidray.on_init(init_game)
    voidray.start()

if __name__ == "__main__":
    main()
```

If you see a window with a blue square, VoidRay is working correctly!

## 🐛 Troubleshooting

### Common Issues

**"pygame not found"**:
```bash
pip install pygame
```

**"Module not found" errors**:
- Ensure you're using the correct Python version
- Try: `python -m pip install voidray`

**Permission errors on macOS/Linux**:
```bash
pip install --user voidray
```

**Display issues on Linux**:
```bash
sudo apt-get install python3-dev python3-pygame
```

### Performance Issues

If you experience slow performance:
1. Update your graphics drivers
2. Ensure hardware acceleration is enabled
3. Close other graphics-intensive applications

## 🎯 Next Steps

Now that VoidRay is installed:

1. **Try the [Quick Start Guide](quick_start.md)** for your first game
2. **Explore the [examples](examples.md)** to see what's possible
3. **Read the [Engine Overview](engine_overview.md)** to understand the architecture

Happy coding! 🚀
