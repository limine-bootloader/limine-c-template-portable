# Nuke built-in rules and variables.
override MAKEFLAGS += -rR

# This is the name that our final kernel executable will have.
# Change as needed.
override KERNEL := kernel.elf

# Convenience macro to reliably declare user overridable variables.
define DEFAULT_VAR =
    ifeq ($(origin $1),default)
        override $(1) := $(2)
    endif
    ifeq ($(origin $1),undefined)
        override $(1) := $(2)
    endif
endef

# Target architecture to build for. Default to x86_64.
override DEFAULT_ARCH := x86_64
$(eval $(call DEFAULT_VAR,ARCH,$(DEFAULT_ARCH)))

# It is highly recommended to use a custom built cross toolchain to build a kernel.
# We are only using "cc" as a placeholder here. It may work by using
# the host system's toolchain, but this is not guaranteed.
override DEFAULT_CC := cc
$(eval $(call DEFAULT_VAR,CC,$(DEFAULT_CC)))

override CC_IS_CLANG := no
ifeq ($(shell $(CC) --version 2>&1 | grep -i 'clang' >/dev/null 2>&1 && echo 1),1)
    override CC_IS_CLANG := yes
endif

# Same thing for "ld" (the linker).
override DEFAULT_LD := ld
$(eval $(call DEFAULT_VAR,LD,$(DEFAULT_LD)))

# User controllable C flags.
override DEFAULT_CFLAGS := -g -O2 -pipe
$(eval $(call DEFAULT_VAR,CFLAGS,$(DEFAULT_CFLAGS)))

# User controllable C preprocessor flags. We set none by default.
override DEFAULT_CPPFLAGS :=
$(eval $(call DEFAULT_VAR,CPPFLAGS,$(DEFAULT_CPPFLAGS)))

ifeq ($(ARCH), x86_64)
    # User controllable nasm flags.
    override DEFAULT_NASMFLAGS := -F dwarf -g
    $(eval $(call DEFAULT_VAR,NASMFLAGS,$(DEFAULT_NASMFLAGS)))
endif

# User controllable linker flags. We set none by default.
override DEFAULT_LDFLAGS :=
$(eval $(call DEFAULT_VAR,LDFLAGS,$(DEFAULT_LDFLAGS)))

# Internal C flags that should not be changed by the user.
override CFLAGS += \
    -Wall \
    -Wextra \
    -std=gnu11 \
    -ffreestanding \
    -fno-stack-protector \
    -fno-stack-check \
    -fno-lto \
    -fPIE

# Internal C preprocessor flags that should not be changed by the user.
override CPPFLAGS := \
    -I. \
    $(CPPFLAGS) \
    -MMD \
    -MP

ifeq ($(ARCH),x86_64)
    # Internal nasm flags that should not be changed by the user.
    override NASMFLAGS += \
        -Wall
endif

# Architecture specific internal flags.
ifeq ($(ARCH),x86_64)
    ifeq ($(CC_IS_CLANG),yes)
        override CC += \
            -target x86_64-elf
    endif
    override CFLAGS += \
        -m64 \
        -march=x86-64 \
        -mabi=sysv \
        -mno-80387 \
        -mno-mmx \
        -mno-sse \
        -mno-sse2 \
        -mno-red-zone
    override LDFLAGS += \
        -m elf_x86_64
    override NASMFLAGS += \
        -f elf64
else ifeq ($(ARCH),aarch64)
    ifeq ($(CC_IS_CLANG),yes)
        override CC += \
            -target aarch64-elf
    endif
    override CFLAGS += \
        -mgeneral-regs-only
    override LDFLAGS += \
        -m aarch64elf
else ifeq ($(ARCH),riscv64)
    ifeq ($(CC_IS_CLANG),yes)
        override CC += \
            -target riscv64-elf
        override CFLAGS += \
            -march=rv64imac
    else
        override CFLAGS += \
            -march=rv64imac_zicsr_zifencei
    endif
    override CFLAGS += \
        -mabi=lp64
    override LDFLAGS += \
        -m elf64lriscv \
        --no-relax
else
    $(error Architecture $(ARCH) not supported)
endif

# Internal linker flags that should not be changed by the user.
override LDFLAGS += \
    -nostdlib \
    -static \
    -pie \
    --no-dynamic-linker \
    -z text \
    -z max-page-size=0x1000 \
    -T linker-$(ARCH).ld

# Use "find" to glob all *.c, *.S, and *.asm files in the tree and obtain the
# object and header dependency file names.
override CFILES := $(shell find -L . -type f -name '*.c')
override ASFILES := $(shell find -L . -type f -name '*.S')
override NASMFILES := $(shell find -L . -type f -name '*.asm')
override OBJ := $(CFILES:.c=.c.o) $(ASFILES:.S=.S.o) $(NASMFILES:.asm=.asm.o)
override HEADER_DEPS := $(CFILES:.c=.c.d) $(ASFILES:.S=.S.d)

# Default target.
.PHONY: all
all: $(KERNEL)

limine.h:
	curl -Lo $@ https://github.com/limine-bootloader/limine/raw/trunk/limine.h

# Link rules for the final kernel executable.
$(KERNEL): $(OBJ)
	$(LD) $(OBJ) $(LDFLAGS) -o $@

# Include header dependencies.
-include $(HEADER_DEPS)

# Compilation rules for *.c files.
%.c.o: %.c limine.h
	$(CC) $(CFLAGS) $(CPPFLAGS) -c $< -o $@

# Compilation rules for *.S files.
%.S.o: %.S limine.h
	$(CC) $(CFLAGS) $(CPPFLAGS) -c $< -o $@

# Compilation rules for *.asm (nasm) files.
%.asm.o: %.asm
	nasm $(NASMFLAGS) $< -o $@

# Remove object files and the final executable.
.PHONY: clean
clean:
	rm -rf $(KERNEL) $(OBJ) $(HEADER_DEPS)

.PHONY: distclean
distclean: clean
	rm -f limine.h
