
# ---- Generic Testbenches ----

testbench: testbench.vvp firmware.hex
	vvp -N $<

testbench.vvp: spiflash.v spimemio.v simpleuart.v testbench.v picosoc.v ../picorv32.v
	iverilog -s testbench -o $@ $^

spiflash_tb: spiflash_tb.vvp firmware.hex
	vvp -N $<

spiflash_tb.vvp: spiflash.v spiflash_tb.v
	iverilog -s testbench -o $@ $^

# ---- iCE40 HX8K Breakout Board ----

hx8kprog: hx8kdemo.bin firmware.bin
	iceprog hx8kdemo.bin
	iceprog -o 1M firmware.bin

hx8ksim: hx8kdemo_tb.vvp firmware.hex
	vvp -N $<

hx8ksynsim: hx8kdemo_syn_tb.vvp firmware.hex
	vvp -N $<

hx8kdemo.blif: hx8kdemo.v spimemio.v simpleuart.v picosoc.v ../picorv32.v
	yosys -ql hx8kdemo.log -p 'synth_ice40 -top hx8kdemo -blif hx8kdemo.blif' $^

hx8kdemo_tb.vvp: hx8kdemo_tb.v hx8kdemo.v spimemio.v simpleuart.v picosoc.v ../picorv32.v spiflash.v
	iverilog -s testbench -o $@ $^ `yosys-config --datdir/ice40/cells_sim.v`

hx8kdemo_syn_tb.vvp: hx8kdemo_tb.v hx8kdemo_syn.v spiflash.v
	iverilog -s testbench -o $@ $^ `yosys-config --datdir/ice40/cells_sim.v`

hx8kdemo_syn.v: hx8kdemo.blif
	yosys -p 'read_blif -wideports hx8kdemo.blif; write_verilog hx8kdemo_syn.v'

hx8kdemo.asc: hx8kdemo.pcf hx8kdemo.blif
	arachne-pnr -d 8k -o hx8kdemo.asc -p hx8kdemo.pcf hx8kdemo.blif

hx8kdemo.bin: hx8kdemo.asc
	icetime -d hx8k -c 12 -mtr hx8kdemo.rpt hx8kdemo.asc
	icepack hx8kdemo.asc hx8kdemo.bin

# ---- Example Firmware ----

firmware.elf: sections.lds firmware.S
	riscv32-unknown-elf-gcc -Wl,-Bstatic,-T,sections.lds,--strip-debug -ffreestanding -nostdlib -o firmware.elf firmware.S

firmware_up.elf: firmware.elf
	riscv32-unknown-elf-objcopy --change-addresses 0x100000 firmware.elf firmware_up.elf

firmware.hex: firmware_up.elf
	riscv32-unknown-elf-objcopy -O verilog firmware_up.elf firmware.hex

firmware.bin: firmware.elf
	riscv32-unknown-elf-objcopy -O binary firmware.elf firmware.bin

# ---- Clean ----

clean:
	rm -f testbench.vvp testbench.vcd spiflash_tb.vvp spiflash_tb.vcd
	rm -f firmware.elf firmware_up.elf firmware.hex firmware.bin
	rm -f hx8kdemo.blif hx8kdemo.log hx8kdemo.asc hx8kdemo.rpt hx8kdemo.bin
	rm -f hx8kdemo_syn.v hx8kdemo_syn_tb.vvp hx8kdemo_tb.vvp

.PHONY: testbench spiflash_tb hx8kprog hx8ksim hx8ksynsim clean

