simavr_sap_emulator - Atmel AVR atmega328p simulator for FIT SAP LCD 1602 keypad board
======

Precompiled simulator & tools for macOS arm64 and Linux x86_64 [here](https://github.com/EETagent/simavr-sap/releases)

<img width="631" alt="image" src="https://github.com/EETagent/simavr-sap-experiment/assets/20557318/ef75870d-84ce-4088-942b-073e04b58874">

## Compiling ASM code using avra (avrasm2 is also supported)

```sh
avra program.asm
```

### Example program

Please only define .org 0 and always include pritnlib.inc after

```asm
.include "asm/m328Pdef.inc"

; Zacatek programu - po resetu
.org 0
    jmp start

; podprogramy pro praci s displejem
.include "asm/printlib.inc"

; Zacatek programu - hlavni program
start:
    ; Inicializace displeje
    call init_disp
    
    ; *** ZDE muzeme psat nase instrukce
    ldi r16, '0'    ; znak
    ldi r17, 0      ; pozice (0x00-0x0f - prvni radek; 0x40-0x4f - druhy radek)
    call show_char  ; zobraz znak z R16 na pozici z R17

end: jmp end        ; Zastavime program - nekonecna smycka
```

## Running the simulator

```sh
simavr_sap_emulator program.hex
```

## VSCode bindings

.vscode/tasks.json

CMD + SHIFT + B now builds active .asm file


```json
{
    "version": "2.0.0",
    "tasks": [
      {
        "label": "Assembler",
        "type": "shell",
        "command": "avra ${file}",
        "problemMatcher": {
          "base": "$gcc",
          "fileLocation": [
            "relative",
          ]
        },
        "group": {
          "kind": "build",
          "isDefault": true
        }
      },
      {
        "label": "Emulate",
        "type": "shell",
        "command": "simavr_sap_emulator ${fileDirname}/${fileBasenameNoExtension}.hex",
        "problemMatcher": [],
        "group": {
          "kind": "test",
          "isDefault": true
        }
      }
    ]
  }
```

## GDB

```sh
simavr_sap_emulator program.hex -d
```

```sh
avr-gdb -q -n -ex 'target remote 127.0.0.1:1234'
```

### GDB Dashboard

<img width="1091" alt="gdb Dashboard" src="https://github.com/EETagent/simavr-sap/assets/20557318/ceb9eec7-b2b9-4904-8a74-2911b335f9b3">

### Setup

```sh
wget -P ~ https://github.com/cyrus-and/gdb-dashboard/raw/master/.gdbinit
# without -n and .gdbinit in $home or (gdb) source .gdbinit at runtime
avr-gdb -q -ex 'target remote 127.0.0.1:1234'
```

#### Run dashboard

```gdb
(gdb) dashboard
```

#### Print next 200 instructions -> print assembly

```gdb
(gdb) x/200i $pc
```

#### Breakpoint

```gdb
(gdb) break *0x1b8
```

#### Finding your code instruction

you can set an additional jump to code part you want to breakpoint. After initializing the screen and using the previous code to dump asm, find the instruction where brk starts. There won't be much jmp instructions in the code. 

```diff
diff --git a/sap/program.asm b/sap/program.asm
index 2e1070f..9b591f1 100644
--- a/sap/program.asm
+++ b/sap/program.asm
@@ -11,7 +11,8 @@
 start:
     ; Inicializace displeje
     call init_disp
-    
+    jmp brk
+brk:
     ; *** ZDE muzeme psat nase instrukce
     ldi r16, '0'    ; znak
     ldi r17, 0      ; pozice (0x00-0x0f - prvni radek; 0x40-0x4f - druhy radek)

```

#### Continue until breakpoint

```gdb
(gdb) c
```

#### Next instruction

```gdb
(gdb) ni
```

## macOS

### Building

```sh
brew tap osx-cross/avr
brew install avr-gcc@12
```

```sh
make
```

### Testing

```sh
cd sap
make
make run

```

### macOS arm64 avr-gdb build

https://mirrors.ocf.berkeley.edu/gnu/gdb/ (latest is 14.2 atm)

```sh
./configure --target=avr --prefix=/Users/xxx/bin/avr-gdb/ --disable-debug  --disable-dependency-tracking --disable-binutils --disable-nls  --disable-libssp --disable-install-libbfd --disable-install-libiberty --with-python=/opt/homebrew/opt/python@3.12/bin/python3.12 --with-gmp=/opt/homebrew/opt/gmp/ --with-mpfr=/opt/homebrew/opt/mpfr/
```

### Tools

you can use the included static programs in the zip

```sh
brew install avrdude avra
```

## Linux

### Building

```sh
sudo apt-get install gcc-avr avr-libc
```

```sh
make
```

### Testing

```sh
cd sap
make
make run
```

### Tools

```sh
sudo apt-get install avrdude avra
```

## More Info

Check [README.simavr.md](https://github.com/EETagent/simavr-sap-experiment/blob/master/README.simavr.md) for more info on the simulator
