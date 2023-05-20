# 外星人入侵

使用`pygame`开发的一个 2d 游戏

`pygame`可用于管理图形、动画乃至声音，可让开发者轻松开发复杂的游戏。通过使用 pygame 来处理在屏幕上绘制图像等任务，不用考虑容多繁琐而艰难的编码工作，而是将重点放在程序的高级逻辑上

## 飞船篇

### 创建游戏窗口

首先创建一个空的 pygame 窗口，供后面用来绘制游戏元素

```py
> cat alien_invasion.py
import sys
import pygame

def run_game():
    """初始化游戏并创建一个屏幕对象"""
    pygame.init() # 初始化背景设置,让 pygame 能正确工作
    screen = pygame.display.set_mode((1200,800))
    pygame.display.set_caption("Alien Invasion")

    #开始游戏的主循环
    while True:
        #监听键盘和鼠标事件
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                sys.exit()

        #让最近绘制的屏幕可见
        pygame.display.flip()

run_game()
```

玩家退出时，使用模块 `sys` 来退出游戏。

对象 screen 是一个 `surface`。在 pygame 中，`surface` 是屏幕的一部分，用于显示游戏元素。在这个游戏游戏中，每个元素(eg：外星人或飞船) 都是一个 `surface`。`display.set_mode()`返回的 surface 表示整个游戏窗口。每次循环都会重绘这个 surface


while 循环内包含一个`事件循环以及管理屏幕更新的代码`。事件由用户触发(eg：案件或移动鼠标)。为了让程序响应时就爱你，我们编写一个事件循环，以监听事件，并根据发生的事件执行相应的任务

`pygame.event.get()`会监听键盘或鼠标的事件，事件会促使 for 循环执行。

`pygame.display.flip()` 命令 Pygame 让最近绘制的屏幕可见。在这里，它在每次执行 while 循环时都绘制一个空屏幕，并擦除旧屏幕，使得只有新屏幕可见。后续在移动游戏元素时，`pygame.display.flip() 将不断更新屏幕，以显示元素的新位置，并在原来的位置隐藏元素，从而营造平滑移动的效果`

#### 设置背景色

pygame 默认创建一个黑色屏幕，现在来设置背景色

```py
> cat alien_invasion.py
import sys
import pygame

def run_game():
    """初始化游戏并创建一个屏幕对象"""
    pygame.init()
    screen = pygame.display.set_mode((1200,800))
    pygame.display.set_caption("Alien Invasion")

    #设置背景色
    bg_color = (230,230,230) #---------新增

    #开始游戏的主循环
    while True:
        #监听键盘和鼠标事件
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                sys.exit()

        #每次循环时都重绘屏幕
        screen.fill(bg_color) #---------新增
        #让最近绘制的屏幕可见
        pygame.display.flip()

run_game()
```

在 pygame 中，颜色以 RGB 指定，取值为 0～255

通过调用 `screen.fill()`，用背景色填充屏幕，这个方法只接收一个实参：一种颜色

#### 创建设置类

每次给游戏添加新功能时，通常也将引入一些新设置。现在来编写一个名为 `settings` 的模块，其中包含一个名为 `Settings` 的类，用于将所有设置存储在一个地方。

```py
> cat settings.py
class Settings():
    """存储《外星人入侵》的所有设置的类"""
    def __init__(self):
        """初始化游戏的设置"""
        #屏幕设置
        self.screen_width = 1200
        self.screen_height = 800
        self.bg_color = (230,230,230)
```

为创建 `Settings` 实例并使用它来访问设置，将 `alien_invasion.py` 修改

```py
> cat alien_invasion.py
import sys
import pygame

from settings import Settings

def run_game():
    """初始化游戏并创建一个屏幕对象"""
    pygame.init()
    ai_settings = Settings() #------------新增
    screen = pygame.display.set_mode(
        (ai_settings.screen_width,ai_settings.screen_height)
    ) #------------新增
    pygame.display.set_caption("Alien Invasion")

    #开始游戏的主循环
    while True:
        #监听键盘和鼠标事件
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                sys.exit()

        #每次循环时都重绘屏幕
        screen.fill(ai_settings.bg_color)
        #让最近绘制的屏幕可见
        pygame.display.flip()

run_game()
```

### 添加飞船图像

为了在屏幕上绘制玩家的飞船，需要加载一副图像，再使用 pygame 的方法 `blit()` 绘制它

在游戏中几乎可以使用任何类型的图像文件，但使用`位图(.bmp)`文件最简单，因为 pygame 默认加载位图

我们将飞船的图片移动到 ./images 下，并重命名为 ship.bmp

#### 创建 Ship 类

选择飞船图像后，需要将其显示到屏幕上。我们创建一个名为 ship 的模块，其中包含 Ship 类，它负责管理飞船的大部分行为

```py
> cat ship.py
import pygame

class Ship():
    def __init__(self, screen):
        """初始化飞船并设置其初始位置"""
        self.screen = screen

        # 加载飞船图像并获取其外接矩形
        self.image = pygame.image.load("images/ship.bmp") #加载图像
        self.rect = self.image.get_rect()
        self.screen_rect = screen.get_rect()

        # 将每艘新飞船放在屏幕底部中央
        self.rect.centerx = self.screen_rect.centerx
        self.rect.bottom = self.screen_rect.bottom

    def blitme(self):
        """在指定位置绘制飞船"""
        self.screen.blit(self.image,self.rect)
```

`Ship 的方法 __init__` 接收的 screen 参数表示：要将飞船绘制到什么地方

`pygame.image.load()` 方法返回一个表示飞船的 `surface`，我们将这个 surface 存储到了 `self.image` 中

`get_rect()` 获取相应 surface 的属性 `rect`。pygame 效率之所以高，一个原因是它让你能处理矩形(rect 对象)一样处理游戏元素，即便他们的形状并非举行

处理 `rect` 对象时，可设置相应 rect 对象的属性 `center, centerx 和 centery`。要让游戏元素与屏幕边缘对齐，可使用属性 `top, bottom, left 和 right`。要调整游戏元素的水平或垂直距离，可使用属性 x 和 y。他们分别是`相应矩形左上角的 x 和 y 坐标`

目前我们将飞船放在了屏幕底部中央。而我们定义的方法`Ship.blitme()` 会根据 `self.rect` 指定的位置将图像绘制到屏幕上

#### 在屏幕上绘制飞船

现在更新 alien_invasion.py，使其创建一艘飞船并显示在屏幕上

```py
> cat alien_invasion.py
import sys
import pygame

from settings import Settings
from ship import Ship

def run_game():
    """初始化游戏并创建一个屏幕对象"""
    pygame.init()
    ai_settings = Settings()
    screen = pygame.display.set_mode(
        (ai_settings.screen_width,ai_settings.screen_height)
    )
    pygame.display.set_caption("Alien Invasion")

    #创建一艘飞船
    ship = Ship(screen) #-------------新增

    #开始游戏的主循环
    while True:
        #监听键盘和鼠标事件
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                sys.exit()

        #每次循环时都重绘屏幕
        screen.fill(ai_settings.bg_color)
        ship.blitme()   #-------------新增
        #让最近绘制的屏幕可见
        pygame.display.flip()

run_game()
```

现在运行 alien_invasion.py。可以看到飞船位于空游戏屏幕底部中央 

![shipAtBottom](../images/et/ship_at_bottom.png) 

### 重构

重构旨在简化既有代码的结构，使其更容易扩展

现在让我们创建一个名为 game_functions 的新模块，它将存储大量让游戏 《外星人入侵》运行的函数。通过创建 game_functions，可避免 alien_invasion.py 文件太常

#### 函数 check_events()

首先把管理时间的代码移到一个名为 `check_events()` 的函数中，以简化 `run_game()`，并隔离事件管理循环。通过隔离事件循环，可将时间管理与游戏的其他方面(eg：更新屏幕) 分离。

将 `check_events()` 放在一个名为 `game_functions` 的模块中

```py
> cat game_functions.py
import sys

import pygame

def check_events():
    """响应按键和鼠标事件"""
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            sys.exit()
```

现在修改 alien_invasion.py，使其导入模块 game_functions，并将时间循环替换为对函数 check_events 的调用

```py
> cat alien_invasion.py
import pygame

from settings import Settings
from ship import Ship
import game_functions as gf

def run_game():
    """初始化游戏并创建一个屏幕对象"""
    pygame.init()
    ai_settings = Settings()
    screen = pygame.display.set_mode(
        (ai_settings.screen_width,ai_settings.screen_height)
    )
    pygame.display.set_caption("Alien Invasion")

    #创建一艘飞船
    ship = Ship(screen)

    #开始游戏的主循环
    while True:
        #监听键盘和鼠标事件
        gf.check_events() #-------------新增

        #每次循环时都重绘屏幕
        screen.fill(ai_settings.bg_color)
        ship.blitme()
        #让最近绘制的屏幕可见
        pygame.display.flip()

run_game()
```

现在，主程序文件中，不需要再导入 sys 模块了

#### 函数 update_screen()

为了进一步简化 `run_game()`，下面将更新屏幕的代码移动到另一个名为 `update_screen()` 的函数中，并将这个函数放在模块 game_functions 中

```py
> cat game_functions.py
import sys

import pygame

def check_events():
    """响应按键和鼠标事件"""
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            sys.exit()

def update_screen(ai_settings,screen,ship): #--------新增
    """更新屏幕的图像，并切换到新屏幕"""
    # 每次循环时都重绘屏幕
    screen.fill(ai_settings.bg_color)
    ship.blitme()

    #让最近重绘的屏幕可见
    pygame.display.flip()
```

`update_screen()` 接收三个参数：`ai_settings, screen, ship`。现在将 alien_invasion.py 的 while 循环中更新屏幕的代码替换为对函数 `update_screen()` 的调用

```py
> cat alien_invasion.py
import sys
import pygame

from settings import Settings
from ship import Ship
import game_functions as gf

def run_game():
    """初始化游戏并创建一个屏幕对象"""
    pygame.init()
    ai_settings = Settings()
    screen = pygame.display.set_mode(
        (ai_settings.screen_width,ai_settings.screen_height)
    )
    pygame.display.set_caption("Alien Invasion")

    #创建一艘飞船
    ship = Ship(screen)

    #开始游戏的主循环
    while True:
        #监听键盘和鼠标事件
        gf.check_events()
        gf.update_screen(ai_settings,screen,ship) #-------------新增

run_game()
```
### 驾驶飞船

在用户按左箭头或右箭头时，应做出响应。

#### 响应按键

每当用户按键时，都会在 pygame 中注册一个时间。事件都是通过方法 `pygame.event.get()` 获取的，因此在 `check_events()` 函数中，我们需要指定：要检查哪些类型的事件。每次按键都被注册为一个 `KEYDOWN` 事件

检测到`KEYDOWN`事件时，我们需要检查按下的是否是特定的键。eg：如果按下的是右箭头键，我们就增大飞船的 `rect.centerx` 值，让飞船向右移动

```py
> cat game_functions.py
--snip--

def check_events(ship):
    """响应按键和鼠标事件"""
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            sys.exit()
        elif event.type == pygame.KEYDOWN: #-------新增
            if event.key == pygame.K_RIGHT:
                #向右移动飞船
                ship.rect.centerx += 1

def update_screen(ai_settings,screen,ship):
--snip--
```

在函数 `check_events()`中传入实参 `ship`，因为玩家按右箭头时，需要将飞船向右移动。在 `check_events()` 内部，我们在事件循环中添加了一个 `elif` 代码块，以便在 pygame 检测到 `KEYDOWN` 事件时做出响应。通过读取 `event.key`，来检查按下的是否是右箭头键(pygame.K_RIGHT)

在 alien_invasion.py 中，我们需要更新调用的 `check_events()` 代码，将 `ship` 作为实参传递给他

```py
--snip--
#开始游戏的主循环
while True:
│   #监听键盘和鼠标事件
│   gf.check_events(ship) #-------------新增
│   gf.update_screen(ai_settings,screen,ship)

--snip--
```

现在运行 alien_invasion.py，每按下右箭头一次，飞船都将向右移动一个像素。

#### 允许不断移动

玩家按住右箭头不放开时，我们希望飞船不断地向右移动，直到玩家松开为止。可以让游戏检测 `pygame.KEYUP` 事件。然后，我们将结合使用`KEYDOWN` 和 `KEYUP` 事件，以及一个名为 `moving_right` 的标志来实现持续移动

飞船不动时，标志 `moving_right` 将为 False，。玩家按下右箭头时，这个标志为 True，玩家松开后，这个标志重新设置为 False

飞船的属性都由 `Ship` 类控制，因此我们给这个类添加一个名为 `moving_right` 的属性和一个名为 `update()` 的方法。`update()` 检查标志 `moving_right` 的状态，如果是 True,就调整飞船的位置。每当需要调整飞船的位置时，就调用这个方法

```py
> cat ship.py
import pygame

class Ship():
    def __init__(self, screen):
        """初始化飞船并设置其初始位置"""
        self.screen = screen

        # 加载飞船图像并获取其外接矩形
        self.image = pygame.image.load("images/ship.bmp")
        self.rect = self.image.get_rect()
        self.screen_rect = screen.get_rect()

        # 将每艘新飞船放在屏幕底部中央
        self.rect.centerx = self.screen_rect.centerx
        self.rect.bottom = self.screen_rect.bottom

        # 移动标志
        self.moving_right = False #---------新增

    def update(self):    #---------新增
        """根据移动标志调整飞船的位置"""
        if self.moving_right:
            self.rect.centerx += 1

    def blitme(self):
        """在指定位置绘制飞船"""
        self.screen.blit(self.image,self.rect)
```

现在修改 `check_events()`，使其在玩家按下右箭头时将 `moving_right` 设置为 True,并在玩家松开后将 `moving_right` 设置为 False

```py
--snip--
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            sys.exit()
        elif event.type == pygame.KEYDOWN:
            if event.key == pygame.K_RIGHT:
                ship.moving_right = True    #---------新增
        elif event.type == pygame.KEYUP:    #---------新增
            if event.key == pygame.K_RIGHT: #---------新增
                ship.moving_right = False   #---------新增

def update_screen(ai_settings,screen,ship):
--snip--
```

最后，我们需要修改 `alien_invasion.py` 中的 while 循环，以便每次执行循环时都调用飞船的方法 update()

```py
--snip--
    #开始游戏的主循环
    while True:
        gf.check_events(ship)
        ship.update() #-------------新增
        gf.update_screen(ai_settings,screen,ship)

run_game()
--snip--
```

现在运行 alien_invasion.py，并按住右箭头，飞船将不断地向右移动，直到松开为止

#### 左右移动

飞船能够不断向右移动后，添加向左移动的逻辑很容易。我们再次修改 Ship 类和函数 check_events()。

```py
> cat ship.py
--snip--
        # 移动标志
        self.moving_right = False
        self.moving_left = False  #-----------新增

    def update(self):
        """根据移动标志调整飞船的位置"""
        if self.moving_right:
            self.rect.centerx += 1
        if self.moving_left:  #-----------新增
            self.rect.centerx -= 1
--snip--
```

之所以添加 if 而不是 elif,这样如果玩家同时按下了左右箭头，将先曾大飞船的 rect.centerx 值，再降低这个值，即飞船的位置保持不变

如果使用 elif 来处理向左移动的情况，右箭头将始终处于优先地位。

现在对 check_events() 做两方面的调整

```py
> cat game_functions
--snip--
        elif event.type == pygame.KEYDOWN:
            if event.key == pygame.K_RIGHT:
                ship.moving_right = True
            elif event.key == pygame.K_LEFT: #-----------新增
                ship.moving_left = True      #-----------新增
        elif event.type == pygame.KEYUP:
            if event.key == pygame.K_RIGHT:
                ship.moving_right = False
            elif event.key == pygame.K_LEFT: #-----------新增
                ship.moving_left = False     #-----------新增
--snip--
```

这里之所以使用 elif，是因为每个事件都只与一个键相关联。如果玩家同时按下了左右箭头，将检测到两个不同的事件

此时如果运行 alien_invasion.py，将能不断地左右移动飞船，如果同时按左右箭头，飞船不会移动

下面进一步优化飞船的移动方式：调整飞船的速度、限制飞船的移动距离，以免它移动到屏幕外围

#### 调整飞船的速度

当前，每次执行 while 循环时，飞船最多移动 1 个像素，但我们可以在 `Settings` 类中添加属性 `ship_speed_factor`，用于控制飞船的速度。

我们将根据这个属性决定飞船在每次循环时最多移动多少距离。下面实在 settings.py 中添加这个属性

```py
> cat settings.py
--snip--
        self.screen_height = 800
        self.bg_color = (230,230,230)
        # 飞船的位置
        self.ship_speed_factor = 1.5 #-------------新增
```

将 `ship_speed_factor` 的初始值设置为 1.5。需要移动飞船时，会移动 1.5 像素而不是 1 像素

通过将速度设置为小树枝，可在后面加快游戏的节奏时更细致地控制飞船的速度。然而，`rect 的 centerx` 等属性只能存储整数，因此我们需要修改 `Ship` 类

```py
> cat ship.py
--snip--
class Ship():
    def __init__(self, ai_settings,screen):  #-----------新增
        """初始化飞船并设置其初始位置"""
        self.screen = screen
        self.ai_settings = ai_settings  #-----------新增

        # 加载飞船图像并获取其外接矩形
        self.image = pygame.image.load("images/ship.bmp")
        self.rect = self.image.get_rect()
        self.screen_rect = screen.get_rect()

        # 将每艘新飞船放在屏幕底部中央
        self.rect.centerx = self.screen_rect.centerx
        self.rect.bottom = self.screen_rect.bottom

        # 在飞船的属性 center 中存储浮点数
        self.center = float(self.rect.centerx)  #-----------新增
        # 移动标志
        self.moving_right = False
        self.moving_left = False

    def update(self):
        """根据移动标志调整飞船的位置"""
        # 更新飞船的 center 值，而不是 rect
        if self.moving_right:
            self.center += self.ai_settings.ship_speed_factor  #-----------新增
        if self.moving_left:
            self.center -= self.ai_settings.ship_speed_factor  #-----------新增

        # 根据 self.center 更新 rect 对象
        self.rect.centerx = self.center  #-----------新增
--snip--
```

在 `__init__()` 的型参中添加了 `ai_settings`，让飞船能够获取其速度设置。

在 alien_invasion.py 中创建 Ship 实例时，需要传入实参 ai_settings

```py
> cat alien_invasion.py
--snip--
    #创建一艘飞船
    ship = Ship(ai_settings,screen)
--snip--
```

现在，只要 `ship_speed_factor` 的值大于 1,飞船的移动速度就会比以前更快。这有助于让飞船的反应速度足够快。

#### 限制飞船的活动范围

当前，如果玩家按住箭头的时间足够长，飞船将移动到屏幕外，下面来修复这个问题，让飞船到达屏幕边缘后停止移动。为此，我们将修改 `Ship` 类的方法 `update()`

```py
> cat ship.py
--snip--
    def update(self):
        """根据移动标志调整飞船的位置"""
        # 更新飞船的 center 值，而不是 rect
        if self.moving_right and self.rect.right < self.screen_rect.right:    #----------新增
            self.center += self.ai_settings.ship_speed_factor
        if self.moving_left and self.rect.left > 0:    #----------新增
            self.center -= self.ai_settings.ship_speed_factor
--snip--
```

上述代码在修改 `self.center` 的值之前先检查飞船的位置。`self.rect.right` 返回飞船外接矩形的右边缘的 x 坐标，如果这个值小于 `self.screen_rect.right` 的值，说明飞船未触及屏幕右边缘。左边缘的情况与此类似

如果 rect 的左边缘的 x 坐标大于0,说明飞船未触及屏幕做边缘。这确保仅当飞船在屏幕内时，才调整 `self.center` 的值

此时运行 alien_invasion.py，飞船将在触及屏幕做边缘或右边缘后停止移动

#### 重构 check_events()

随着游戏开发的进行，函数 `check_events()` 将越来越长，我们将其部分代码放在两个函数中：一个处理`KEYDOWN`事件，另一个处理`KEYUP`事件

```py
> cat game_functions.py
--snip--
def check_keydown_events(event,ship):      #-----------新增
    """响应按下"""
    if event.key == pygame.K_RIGHT:
        ship.moving_right = True
    elif event.key == pygame.K_LEFT:
        ship.moving_left = True

def check_keyup_events(event,ship):      #-----------新增
    """响应松开"""
    if event.key == pygame.K_RIGHT:
        ship.moving_right = False
    elif event.key == pygame.K_LEFT:
        ship.moving_left = False

def check_events(ship):
    """响应按键和鼠标事件"""
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            sys.exit()
        elif event.type == pygame.KEYDOWN:
            check_keydown_events(event,ship)    #-----------新增
        elif event.type == pygame.KEYUP:
            check_keyup_events(event,ship)      #-----------新增
--snip--
```

我们创建了两个新函数 `check_keydown_events()` 和 `check_keyup_events()`，他们都包含型参 `event` 和 `ship`。这两个函数是从 `check_events()`中复制而来。现在，函数`check_events()`更简单

### 射击

现在来添加射击功能。我们将编写玩家按空格键时发射子弹(小矩形)的带澳门。子弹将在屏幕中向上穿行，抵达屏幕上边缘后消失

#### 添加子弹设置

首先爱你，更新 settings.py，在其方法 __init__ 末尾存储新类 `Bullet` 所需的值

```py
> cat settings.py
class Settings():
    """存储《外星人入侵》的所有设置的类"""
    def __init__(self):
        """初始化游戏的设置"""
        #屏幕设置
        --snip--
        # 飞船的位置
        self.ship_speed_factor = 1.5

        #子弹设置
        self.bullet_speed_factor = 1
        self.bullet_width = 3
        self.bullet_height = 15
        self.bullet_color = (60,60,60)
```

设置了创建宽 3 像素、高 15 像素的深灰色子弹。子弹的速度比飞船稍低

#### 创建 Bullet 类

现在创建存储 `Bullet` 类的文件 `bullet.py`。子弹并非基于图像的，因此需要 `pygame.Rect()`类从空白开始创建一个矩形

```py
> cat bullet.py
import pygame
from pygame.sprite import Sprite

class Bullet(Sprite):
    """一个对飞船发射的子弹进行管理的类"""
    def __init__(self, ai_settings,screen,ship):
        """在飞船所处的位置创建一个子弹对象"""
        super(Bullet,self).__init__() # 本行为 python2.7的语法，python3语法为 super().__init__()
        self.screen = screen

        #在(0,0) 处创建一个表示子弹的矩形，再设置正确的位置
        self.rect = pygame.Rect(0,0,ai_settings.bullet_width,
                                ai_settings.bullet_height)
        self.rect.centerx = ship.rect.centerx
        self.rect.top = ship.rect.top

        #存储用小数表示的子弹位置
        self.y = float(self.rect.y)

        self.color = ai_settings.bullet_color
        self.speed_factor = ai_settings.bullet_speed_factor
```

Bullet 类继承了从模块 `pygame.sprite` 中导入的 `Sprite` 类。通过使用精灵，可将游戏中相关的元素编组，进而同时操作编组中的所有元素

为创建子弹实例，需要向__init__()传递 ai_settings,screen,ship，还调用了 `super()`来继承`Sprite`

下面是 bullet.py 的方法

```py
> cat bullet.py
--snip--
class Bullet(Sprite):
    """一个对飞船发射的子弹进行管理的类"""
    def __init__(self, ai_settings,screen,ship):
        """在飞船所处的位置创建一个子弹对象"""
        --snip--

    def update(self):
        """向上移动子弹"""
        # 更新表示子弹位置的小数值
        self.y -= self.speed_factor
        # 更新表示子弹的 rect 的位置
        self.rect.y = self.y

    def draw_bullet(self):
        """在屏幕上绘制子弹"""
        pygame.draw.rect(self.screen,self.color,self.rect)
```


#### 将子弹存储到编组中

定义 `Bullet` 类和必要的设置后，就可以编写代码，按下空格就发射子弹

首先爱你，在 alien_invasion.py 中创建一个编组(group)，用于存储所有有效的子弹，以便能够管理发射出去的所有子弹。

这个编组是 `pygame.sprite.Group` 类的一个实例。`pygame.sprite.Group` 类似于列表，但提供了有助于开发游戏的额外功能。

在主循环中，我们将使用这个编组在屏幕上绘制子弹，以及更新每颗子弹的位置

```py
> cat alien_invasion.py
--snip--
from pygame.sprite import Group
--snip--
def run_game():
    """初始化游戏并创建一个屏幕对象"""
    --snip--
    #创建一艘飞船
    ship = Ship(ai_settings,screen)
    # 创建一个用于存储子弹的编组
    bullets = Group() #-------------新增

    #开始游戏的主循环
    while True:
        gf.check_events(ai_settings,screen,ship,bullets) #-------------新增
        ship.update()
        bullets.update() #-------------新增
        gf.update_screen(ai_settings,screen,ship,bullets) #-------------新增

run_game()
```

#### 开火

在 game_functions.py 中，我们需要修改 `check_keydown_events()`，以便在玩家按空格时发射一颗子弹。无需修改 `check_keyup_events()`，因为玩家松开空格后什么都不会发生。我们还需要修改 `update_screen()`，确保在调用 `flip()` 前在屏幕上重绘每颗子弹。

```py
> cat game_functions.py
--snip--
def check_keydown_events(event,ai_settings,screen,ship,bullets): 
    """响应按下"""
    if event.key == pygame.K_RIGHT:
        ship.moving_right = True
    elif event.key == pygame.K_LEFT:
        ship.moving_left = True
    elif event.key == pygame.K_SPACE:    #-----------新增
        # 创建一颗子弹，并加入到编组 bullets 中
        new_bullet = Bullet(ai_settings,screen,ship)    #-----------新增
        bullets.add(new_bullet)    #-----------新增
--snip--
def check_events(ai_settings,screen,ship,bullets):
    """响应按键和鼠标事件"""
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            sys.exit()
        elif event.type == pygame.KEYDOWN:
            check_keydown_events(event,ai_settings,screen,ship,bullets)     #-----------新增
        elif event.type == pygame.KEYUP:
            check_keyup_events(event,ship)

def update_screen(ai_settings,screen,ship,bullets):
    """更新屏幕的图像，并切换到新屏幕"""
    # 每次循环时都重绘屏幕
    screen.fill(ai_settings.bg_color)
    # 在飞船和外星人后面重绘所有子弹
    for bullet in bullets:     #-----------新增
        bullet.draw_bullet()     #-----------新增
    ship.blitme()
--snip--
```

现在，已经可以正常发射子弹了

#### 删除消失的子弹

当前，子弹抵达屏幕顶端后消失，这仅仅是因为 pygame 无法在屏幕外绘制它们。这些子弹实际上依然存在，它们的 y 坐标为负，且越来越小。

我们需要删除这些消失的子弹，需要检测这样的条件，即`表示子弹的 rect 的 bottom 属性为 0,就表明子弹已穿过屏幕顶端`

```py
> cat alien_invasion.py
--snip--
    #开始游戏的主循环
    while True:
        gf.check_events(ai_settings,screen,ship,bullets)
        ship.update()
        bullets.update()
        # 删除已消失的子弹
        for bullet in bullets.copy(): #-------------新增
            if bullet.rect.bottom <= 0: #-------------新增
                bullets.remove(bullet) #-------------新增
#        print(len(bullets)) #-------------新增
        gf.update_screen(ai_settings,screen,ship,bullets)
        run_game()
```

在 for 循环中，不应从列表或编组中删除条目，因此必须遍历编组的副本。我们使用方法`copy()`来设置 for 循环，这让我们能够在循环中修改 `bullets`。

#### 限制子弹数量

很多设计游戏都对可同时出现在屏幕上的子弹数量进行限制，以鼓励玩家有目标地射击。

首先，在 settings.py 中存储所允许的最大子弹数

```py
> tail settings.py
--snip--
        self.bullet_color = (60,60,60)
        self.bullets_allowed = 3 # 限制子弹数量
```

这将未消失的子弹数限制为 3 颗。在 game_functions.py 的 check_keydown_events() 中，我们在创建新子弹前检查未消失的子弹数是否小于该设置

```py
> cat game_functions.py
--snip--
def check_keydown_events(event,ai_settings,screen,ship,bullets):
    """响应按下"""
--snip--
    elif event.key == pygame.K_SPACE:
        # 创建新子弹并将其加入到编组 bullets 中
        if len(bullets) < ai_settings.bullets_allowed:      #-----------新增
            # 创建一颗子弹，并加入到编组 bullets 中
            new_bullet = Bullet(ai_settings,screen,ship)
            bullets.add(new_bullet)
--snip--
```

玩家按空格时，我们检查 `bullets` 的长度，如果其长度小于3,我们就创建一个新子弹。但如果已有3颗未消失的子弹，则玩家按空格后什么也不会发生

#### 函数 update_bullets()

编写并检查子弹管理代码后，可将其移动到模块`game_functions`中，以让主程序文件 alien_invasion.py 尽可能简单。我们创建一个名为 `update_bullets()` 的新函数，将其添加到 game_functions.py 的末尾

```py
> cat game_functions.py
--snip--
def update_bullets(bullets):
    """更新子弹的位置，并删除已消失的子弹"""
    # 更新子弹的位置
    bullets.update()

    # 删除已消失的子弹
    for bullet in bullets.copy():
        if bullet.rect.bottom <= 0:
            bullets.remove(bullet)
#        print(len(bullets))
```

update_bullets() 的代码均从 alien_invasion.py 剪切而来

alien_invasion.py 文件更新如下

```py
> cat alien_invasion.py
--snip--
    #开始游戏的主循环
    while True:
        gf.check_events(ai_settings,screen,ship,bullets)
        ship.update()
        gf.update_bullets(bullets) #------------新增
        gf.update_screen(ai_settings,screen,ship,bullets)
--snip--
```

#### 函数 fire_bullet()

下面将发射子弹的代码移动到一个独立的函数中，这样，在 `check_keydown_events()` 中只需要一行代码来发射子弹

```py
> cat game_functions.py
--snip--
    elif event.key == pygame.K_SPACE:
        fire_bullet(ai_settings,screen,ship,bullets) #-------------新增

def fire_bullet(ai_settings,screen,ship,bullets): #-------------新增
    """如果还没到达限制，就发射一颗子弹"""
    # 创建新子弹并将其加入到编组 bullets 中
    if len(bullets) < ai_settings.bullets_allowed:
        new_bullet = Bullet(ai_settings,screen,ship)
        bullets.add(new_bullet)
--snip--
```

函数 `fire_bullet()` 只包含玩家按空格时用于发射子弹的代码。

## 外星人篇
