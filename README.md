# Prerequisites

- Python 3.6 (For linux users can be most recent python version)

## How to use it
There's 3 variables to modify:

**BIND_KEY** - The keyboard button name that going to enable/disable the auto clicker

**CLICK_PER_SECONDS** - How many times the button need to be pressed in a second.

**BIND_MOUSE_BUTTON** - What's the button the auto clicker is going to press. 1 - Left | 2 - Right | 3 - Middle

### Windows users
```powershell
cd /your/cloned/folder/directory
run_application.bat
```

### Linux users
```bash
cd /your/cloned/folder/directory
chmod +x run_application.sh
./run_application
```

## Python script
```python
# The keyboard key that's enable/disable the auto clicker
BIND_KEY = 'F12'

# How many clicks will do in a second
CLICK_PER_SECONDS = 1

# What's the button that going to be pressed: 1 - Left | 2 - Right | 3 - Middle
BIND_MOUSE_BUTTON = 2

import logging
import os
import subprocess
import sys
import time
from threading import Thread

is_windows = sys.platform.startswith('win')

try:
    from pykeyboard import PyKeyboardEvent
    from pymouse import PyMouse
except ModuleNotFoundError:
    __install_dependencies = (lambda name:
                              subprocess.Popen(args=['python', '-m', 'pip', 'install', *name.split(' ')]).communicate())

    __install_dependencies('--upgrade pip')
    if is_windows:
        __install_dependencies('pywin32')
        abs_path = os.path.join(os.path.dirname(__file__), 'win32', 'pyHook-1.5.1-cp36-cp36m-win32.whl')
        __install_dependencies(abs_path)
    else:
        __install_dependencies('Xlib')

    __install_dependencies('PyUserInput')
finally:
    from pykeyboard import PyKeyboardEvent
    from pymouse import PyMouse


class AutoClicker(PyKeyboardEvent):
    def __init__(self):
        super(AutoClicker, self).__init__()
        self.mouse = PyMouse()

        self.keycode = None if is_windows else self.lookup_character_keycode(BIND_KEY)
        self.enable = False
        self.sleep = 1 / CLICK_PER_SECONDS

        self.logger = self.__create_logger()

    def run(self):
        self.logger.info('AutoClick: Started')
        super(AutoClicker, self).run()

    def tap(self, keycode, character, press):
        if press and (keycode == self.keycode or character == BIND_KEY):
            self.enable = not self.enable
            if self.enable:
                self.logger.info('AutoClick: Enabled')
                Thread(target=self.__click).start()
            else:
                self.logger.info('AutoClick: Disabled')

    def __click(self):
        while self.enable:
            x, y = self.mouse.position()
            self.mouse.click(x, y, button=BIND_MOUSE_BUTTON)
            if self.enable:
                time.sleep(self.sleep)

    @staticmethod
    def __create_logger():
        logger = logging.getLogger()
        logger.setLevel(logging.INFO)

        handler = logging.StreamHandler(sys.stdout)
        handler.setLevel(logging.INFO)
        formatter = logging.Formatter('%(asctime)s - %(message)s')
        handler.setFormatter(formatter)
        logger.addHandler(handler)
        return logger


AutoClicker().run()
```
