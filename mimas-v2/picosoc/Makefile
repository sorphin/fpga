name = picosoc
top  = rtl/top.v
rtl  = rtl/soc/picosoc.v rtl/soc/picorv32.v rtl/peripheral/uart.v rtl/peripheral/vga.v rtl/util/pll.v rtl/util/fifo.v
ucf  = mimas.ucf
src  = src/main.c src/uart.c src/sevenseg.c src/vga.c

DEVICE = xc6slx9-2csg324
RV_PRE = riscv32-unknown-elf
CFLAGS = 

all: clean image

core: build/$(name)_core.bit
firmware: build/$(name)_firmware.elf

build/$(name)_core.bit:
	@printf "\n====== Building RISC-V CPU core ======\n"
	mkdir -p build
	mkdir build/netlist
	cd build; xst -ifn ../xilinx/picosoc.xst -ofn $(name).srp
	cp xilinx/vga_bram.ngc build/netlist/vga_bram.ngc
	cd build; ngdbuild -p ${DEVICE} -bm ../xilinx/picosoc.bmm -uc ../$(ucf) -sd netlist netlist/picosoc_core.ngc picosoc_core.ngd
	cd build; map -w -timing -detail -p ${DEVICE} -o picosoc_core_map.ncd picosoc_core.ngd picosoc.pcf
	cd build; par -w picosoc_core_map.ncd picosoc_core.ncd picosoc.pcf
	cd build; bitgen -w picosoc_core.ncd $(name)_core.bit picosoc.pcf

build/$(name)_firmware.elf:
	@printf "\n====== Building firmware ======\n"
	mkdir -p build
	$(RV_PRE)-gcc -c src/start.S -o build/start.o
	$(RV_PRE)-gcc -ffreestanding -nostdlib -Wl,-Bstatic,-T,src/sections.lds build/start.o $(src) -lgcc -o build/$(name)_firmware.elf 

image: build/$(name)_core.bit build/$(name)_firmware.elf
	@printf "\n====== Assembling bitstream ======\n"
	data2mem -bm xilinx/picosoc_bd.bmm -bd build/$(name)_firmware.elf -bt build/$(name)_core.bit -o b build/$(name).bit
	promgen -w -b -p bin -u 0 build/$(name).bit -o build/$(name).bin
	cp build/$(name).bin $(name).bin

test:
	@printf "\n====== Running testbench ======\n"
	iverilog -o test/tb test/tb.v rtl/peripheral/sevseg.v
	vvp -n test/tb > /dev/null
	gtkwave test/tb.vcd --rcvar 'fontname_signals Monospace 22' --rcvar 'fontname_waves Monospace 12'

clean:
	@printf "\n====== Cleaning development environment ======\n"
	rm -rf build
	rm -f $(name).bin
	rm -f test/tb test/tb.vcd

.PHONY: all core firmware image test clean
