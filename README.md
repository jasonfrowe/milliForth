# A milliForth for Picocomputer 6502

Picocomputer 6502: https://picocomputer.github.io/

Look upstream for more info: https://github.com/agsb/milliForth-6502

Note that this port uses Picocomputer STDIN. The milliForth docs mention a lack of support for even backspace, but you will have full line editing on a Picocomputer.

I'm able to run `my_hello_world.FORTH` by pasting it through minicom with a 500ms newline tx delay.

To update `sector-6502.s` do two things.

1. Comment out the implementations of getchar, putchar, and byes. Replace with: `.import getchar, putchar, byes`

2. Make sure that `tib_end = $50` is still $50. Currently, milliForth doesn't use this, but the Picocomputer STDIN can be adjusted to make this buffer safe regardless.



## What Was Updated For Current Tooling

This repo was adapted to work with current RP6502/cc65 tooling by aligning the local build helper files with `vscode-cc65`:

- Updated `tools/rp6502.cmake` to the modern cc65 toolchain setup.
- Updated `tools/CMakeLists.txt` to current `rp6502_executable()` behavior (`DATA` address argument).
- Updated `tools/rp6502.py` and added `tools/cc65.cmake` from the newer toolset.
- Updated top-level `CMakeLists.txt` to use:
	- `cmake_minimum_required(VERSION 3.20)`
	- `rp6502_executable(sector6502 DATA 0x300 RESET 0x300)`

To keep `.vscode` machine-independent, local cc65 path setup is done in `CMakeUserPresets.json` instead of hardcoding PATH in workspace settings.

Example local preset:

```json
{
		"version": 3,
		"configurePresets": [
				{
						"name": "rp6502",
						"displayName": "RP6502",
						"toolchainFile": "${sourceDir}/tools/rp6502.cmake",
						"generator": "Unix Makefiles",
						"binaryDir": "${sourceDir}/build",
						"environment": {
								"PATH": "/Users/rowe/Software/rp6502/cc65:$penv{PATH}"
						}
				}
		]
}
```

Build:

```sh
cmake --preset rp6502
cmake --build build
```

## Using screen

`upload` only copies a file to Picocomputer storage. It does not execute Forth source.
To bootstrap milliForth with `my_hello_world.FORTH`, run the ROM and then feed source over serial stdin.

1. Start milliForth ROM (without attaching terminal in `rp6502.py`):

```sh
python3 tools/rp6502.py -c .rp6502 -t false run build/sector6502.rp6502
```

2. Open `screen` on the serial device:

```sh
screen -S pico /dev/cu.usbmodem11401 115200
```

3. From a second terminal, send the Forth file line-by-line with carriage return and delay:

```sh
while IFS= read -r line; do
	screen -S pico -X stuff "$line"
	screen -S pico -X stuff $'\r'
	sleep 0.5
done < milliForth-6502/6502/my_hello_world.FORTH
```

Notes:

- Use carriage return (`\r`), not just newline (`\n`).
- If input overruns, increase delay (for example `0.7`).
- After bootstrap, test with:

```text
5 3 + .
```

4. Exit `screen`: `Ctrl-A`, then `K`.


