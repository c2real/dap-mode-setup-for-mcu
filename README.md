# dap-mode-setup-for-mcu

For debugging MCU, vscode has lots extensions. I wonder how it can work under emacs, then after some search, I find dap-mode package for debugging.

I use dap-gdb-lldb, achieve the goal. The basic idea is using openocd as gdbserver, which is running background, then from emacs/dap-gdb-lldb side, start a arm-none-eabi-gdb session, connecting to the gdbserver, provide the dap-mode to debugging MCU ability, just like vscode extension cortex-debug.

## testing environment
+ OS:Archlinux under VirutalBox
+ Hardware:STM32F429I Discovery

## steps
1. install arm tool chains.
   use the gcc-arm-none-eabi-9-2020-q2-update-x86_64-linux.tar.bz2 from https://developer.arm.com/. 
   arm-none-eabi-gdb need 32bit ncurses library, so also install the AUR ncurses5-compat-libs package.

2. install openocd
3. install nodejs
4. generate stm32 base code project by STM32CubeMX, The project directory is like this.
```
➜  STM32F429-Base git:(main) tree -L 1
.
├── build
├── compile_commands.json
├── Core
├── Drivers
├── gdbstartup
├── launch.json
├── LICENSE
├── Makefile
├── Middlewares
├── startup_stm32f429xx.s
├── STM32F429-Base.ioc
├── STM32F429ZITx_FLASH.ld
└── stm32f4discovery.cfg
```

5. add dap-mode to emacs.
```
(use-package dap-mode
  :ensure t
  :config
  (dap-ui-mode 1)
  (dap-tooltip-mode 1)
  (tooltip-mode 1)
  (dap-ui-controls-mode 1)
  (require 'dap-gdb-lldb)
)

```

6. setup dap-gdb-lldb
```
M-x dap-gdb-lldb-setup
```

7. add launch.json and gdb command file to project.
lanuch.json file
```
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Debug (OpenOCD)",
            "type": "gdb",
            "request": "launch",
            "gdbpath": "arm-none-eabi-gdb",
            "cwd": "/home/c2real/Projects/STM32F429-Base",
            "target": "/home/c2real/Projects/STM32F429-Base/build/STM32F429-Base.elf",
            "debugger_args": "-command=/home/c2real/Projects/STM32F429-Base/gdbstartup",
            "showDevDebugOutput": true,
            "printCalls": true
        }
    ]
}
```
gdbstartup file
```
target remote 127.0.0.1:3333

monitor halt

monitor reset

file /home/c2real/Projects/STM32F429-Base/build/STM32F429-Base.elf

load
```

8. start debugging
when finish these basic setting, then can start debugging.
I also copy the openocd config script from /usr/local/openocd/scripts/board to local direcotry for convenience.

![start openocd](https://github.com/c2real/dap-mode-setup-for-mcu/blob/main/picture/start-openocd.png?raw=true)

Then start emacs.
![start emacs](https://github.com/c2real/dap-mode-setup-for-mcu/blob/main/picture/debugging.gif)

