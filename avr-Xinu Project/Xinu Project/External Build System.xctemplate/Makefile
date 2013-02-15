#
#  Makefile for the avr-Xinu user:
#	___PROJECTNAME___
#  Created by ___FULLUSERNAME___ on ___DATE___.
#
# 1) Set target platform for this project
P = 1284p
# 2) Set root/toplevel directory
TOPDIR    = /Users/Xinu/avr-Xinu
# 3) Add additional files to MAIN_SRC and MAIN_OBJ
MAIN_SRC  = Xinu_main.c # ...
MAIN_OBJ  = Xinu_main.o # ...
# 4) Execute the make file
#		make [P=platform] [DEBUG=YES] target

#############################################################################
# Edit to change default Platform                                           #
# Default (P)latform can be overridden on command line: make P=128 [target] #
#############################################################################
P=1284usb

#############################################################################
# Edit host serial line device (Serial Port to programmer)                  #
#############################################################################
HOST_SERIAL = /dev/cu.usbserial-FTSMQBJS

#############################################################################
# Edit Avrdude programmer flags                                             #
#############################################################################
PROGRAMMER = -c stk500v2 -P $(HOST_SERIAL)

ifeq ($(P),1284p)
PLATFORM := ATmega1284P
TARGET_ARCH := atmega1284p
SYSCOMPS := src/sys/sys src/sys/net src/sys/ENC28J60-tcp src/sys/platforms/$(PLATFORM) src/sys/tty src/sys/shell src/sys/rf
else ifeq ($(P),128)
PLATFORM := ExtendedMemory128
TARGET_ARCH := atmega128
SYSCOMPS := src/sys/sys src/sys/net src/sys/W5300-udp src/sys/platforms/$(PLATFORM) src/sys/shell src/sys/rf src/sys/tty
else ifeq ($(P),162)
PLATFORM := ExtendedMemory162
TARGET_ARCH := atmega162
SYSCOMPS := src/sys/sys src/sys/net src/sys/platforms/$(PLATFORM) src/sys/tty
else ifeq ($(P),1284usb)
PLATFORM := AVR-P40-USB
TARGET_ARCH := atmega1284p
SYSCOMPS := src/sys/sys src/sys/net src/sys/platforms/$(PLATFORM) src/sys/tty
endif

# include Makefile_Vars to get platform specific varables: Fuses, clock frequency, etc.
include ${TOPDIR}/platforms/$(PLATFORM)/Makefile_Vars

AVRDUDE	= avrdude $(PROGRAMMER) -p $(TARGET_ARCH)
COMPILE = avr-gcc
LD = avr-ld
#LIB2 = /usr/local/CrossPack-AVR/avr/lib
#LIB = ~/Software/AVR/Xinu/src/lib
AR = avr-ar
RANLIB = avr-ranlib

ifdef DEBUG
DFLAG	= -DDEBUG
else
DFLAG	=
endif

LIBDIR    = ${TOPDIR}/platforms/$(PLATFORM)/lib
CONFDIR   = ${TOPDIR}/platforms/$(PLATFORM)/h
INCDIR	  =	$(TOPDIR)/src/sys/h

INCLUDE  = -I$(CONFDIR) -I$(INCDIR)

# C compilation flags
CFLAGS =  -Wall -Os -DF_CPU=$(CLOCK) -mmcu=$(TARGET_ARCH) $(INCLUDE) $(DFLAG)

# Assembler flags
ASFLAGS  = -x assembler-with-cpp

# Loader flags
LDFLAGS = -Wl,--wrap,malloc,--wrap,free,--wrap,realloc,--wrap,fdevopen $(EXTERNAL_RAM)

# -ffreestanding prevents optimization from replacing 'printf("no conversions\n");'
#    with 'puts("no conversions\n");' for example,
#    but who knows what else.
#    In one test -ffreestanding was shorter(!) than code generated without it
#    by more than the length of puts().
GCCFLAGS =  -ffreestanding

MSG_ERRORS_NONE = Errors: none
MSG_BEGIN = -------- begin --------
MSG_END = --------  end  --------
MSG_FLASH = Creating load file for Flash:
MSG_EEPROM = Creating load file for EEPROM:
MSG_EXTENDED_LISTING = Creating Extended Listing:
MSG_SYMBOL_TABLE = Creating Symbol Table:
MSG_LINKING = Linking:
MSG_COMPILING = Compiling:
MSG_ASSEMBLING = Assembling:
MSG_CLEANING = Cleaning project:
MSG_CONFIG = Configuring Xinu:

# Compiler flags to generate dependency files.
GENDEPFLAGS = -MD -MP -MF .dep/$(@F).d

CFLAGS += $(GENDEPFLAGS)

LIB_ARC  = ${LIBDIR}/libx.a


#BASE_OBJ = malloc.o getc.o putc.o initialize.o clkint.o
BASE_OBJ = initialize.o clkint.o

# symbolic targets:
all:	$(LIB_ARC) main.hex $(MAIN_OBJ)

.c.o:
	$(COMPILE) $(CFLAGS) $(GCCFLAGS) -c $< -o $@

.S.o:
	$(COMPILE) $(CFLAGS) $(GCCFLAGS) $(ASFLAGS) -c $< -o $@

.s.o:
	$(COMPILE) $(CFLAGS) $(GCCFLAGS) $(ASFLAGS) -c $< -o $@

.c.s:
	$(COMPILE) $(CFLAGS) $(GCCFLAGS) -S $< -o $@

flash install:	all
	$(AVRDUDE) -U flash:w:main.hex:i

# Xcode uses the Makefile targets "", "clean" and "install"
#install: flash fuse

# if you use a bootloader, change the command below appropriately:
#load: all
#	bootloadHID main.hex

clean:
	rm -f main.hex main.elf
	rm -f $(BASE_OBJ) $(MAIN_OBJ)

# Relocatable linking with relocations from format elf32-avr to format elf32-avr is not supported
#xinu.o:	$(XOFILES)
#	$(LD) -i $(XOFILES) -o xinu.o

main.elf: $(LIB_ARC) $(MAIN_OBJ)
	@echo $(DEFS)
	$(AR) x $(LIB_ARC)	$(BASE_OBJ)
	$(COMPILE) $(CFLAGS) $(GCCFLAGS) $(LDFLAGS) -o main.elf $(BASE_OBJ) $(MAIN_OBJ) $(LIB_ARC) -lc

main.hex: main.elf
	rm -f main.hex
	avr-objcopy -j .text -j .data -O ihex main.elf main.hex
	
# Targets for code debugging and analysis:
disasm:	main.elf
	avr-objdump -h -t -d main.elf
	
cref: main.elf
	$(COMPILE) $(CFLAGS) $(GCCFLAGS) $(LDFLAGS) -Wl,--cref $(BASE_OBJ) $(MAIN_OBJ) $(LIB_ARC) -lc

# Create a symbol table from ELF output file.
%.sym: %.elf
	@echo
	@echo $(MSG_SYMBOL_TABLE) $@
	avr-nm -n $< > $@

# Include the dependency files.
-include $(shell mkdir .dep 2>/dev/null) $(wildcard .dep/*)

# Listing of phony targets.
.PHONY : all gccversion install flash fuse config clean disasm cref

