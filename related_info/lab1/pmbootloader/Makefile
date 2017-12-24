all:pmboot
pmboot_mid.s: pmboot.S
	gcc  -E  pmboot.S > pmboot_mid.s
pmboot.o: pmboot_mid.s
	as -o pmboot.o pmboot_mid.s
pmboot: pmboot.o
	ld --oformat binary -N -e start -Ttext 0x7c00 -o pmboot pmboot.o
run: pmboot
	qemu-system-i386 -fda pmboot
.PHONY: clean
clean:
	rm pmboot.o pmboot pmboot_mid.s
