all:pmboot
pmboot.ss: pmboot.S
	gcc  -E  pmboot.S > pmboot.ss
pmboot.o: pmboot.ss
	as -o pmboot.o pmboot.ss
#pmboot.o: pmboot.S
	#as -o pmboot.o pmboot.S
pmboot: pmboot.o
	ld --oformat binary -N -e start -Ttext 0x7c00 -o pmboot pmboot.o
run: pmboot
	qemu-system-i386 -fda pmboot
.PHONY: clean
clean:
	rm pmboot.o pmboot
