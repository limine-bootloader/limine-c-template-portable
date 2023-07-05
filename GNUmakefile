# Nuke built-in rules and variables.
override MAKEFLAGS += -rR

override IMAGE_NAME := barebones

# Convenience macro to reliably declare user overridable variables.
define DEFAULT_VAR =
    ifeq ($(origin $1),default)
        override $(1) := $(2)
    endif
    ifeq ($(origin $1),undefined)
        override $(1) := $(2)
    endif
endef

# Compiler for building the 'limine' executable for the host.
override DEFAULT_HOST_CC := cc
$(eval $(call DEFAULT_VAR,HOST_CC,$(DEFAULT_HOST_CC)))

# Target architecture to build for. Default to x86_64.
override DEFAULT_ARCH := x86_64
$(eval $(call DEFAULT_VAR,ARCH,$(DEFAULT_ARCH)))

.PHONY: all
all: $(IMAGE_NAME).iso

.PHONY: all-hdd
all-hdd: $(IMAGE_NAME).hdd

.PHONY: run
run: run-$(ARCH)

.PHONY: run-hdd
run-hdd: run-hdd-$(ARCH)

.PHONY: run-x86_64
run-x86_64: ovmf $(IMAGE_NAME).iso
	qemu-system-x86_64 -M q35 -m 2G -bios ovmf-x86_64/OVMF.fd -cdrom $(IMAGE_NAME).iso -boot d

.PHONY: run-hdd-x86_64
run-hdd-x86_64: ovmf $(IMAGE_NAME).hdd
	qemu-system-x86_64 -M q35 -m 2G -bios ovmf-x86_64/OVMF.fd -hda $(IMAGE_NAME).hdd

.PHONY: run-aarch64
run-aarch64: ovmf $(IMAGE_NAME).iso
	qemu-system-aarch64 -M virt -cpu cortex-a72 -device ramfb -device qemu-xhci -device usb-kbd -m 2G -bios ovmf-aarch64/OVMF.fd -cdrom $(IMAGE_NAME).iso -boot d

.PHONY: run-hdd-aarch64
run-hdd-aarch64: ovmf $(IMAGE_NAME).hdd
	qemu-system-aarch64 -M virt -cpu cortex-a72 -device ramfb -device qemu-xhci -device usb-kbd -m 2G -bios ovmf-aarch64/OVMF.fd -hda $(IMAGE_NAME).hdd

.PHONY: run-riscv64
run-riscv64: ovmf $(IMAGE_NAME).iso
	qemu-system-riscv64 -M virt -cpu rv64 -device ramfb -device qemu-xhci -device usb-kbd -m 2G -drive if=pflash,unit=1,format=raw,file=ovmf-riscv64/OVMF.fd -device virtio-scsi-pci,id=scsi -device scsi-cd,drive=cd0 -drive id=cd0,format=raw,file=$(IMAGE_NAME).iso

.PHONY: run-hdd-riscv64
run-hdd-riscv64: ovmf $(IMAGE_NAME).hdd
	qemu-system-riscv64 -M virt -cpu rv64 -device ramfb -device qemu-xhci -device usb-kbd -m 2G -drive if=pflash,unit=1,format=raw,file=ovmf-riscv64/OVMF.fd -device virtio-scsi-pci,id=scsi -device scsi-hd,drive=hd0 -drive id=hd0,format=raw,file=$(IMAGE_NAME).hdd

.PHONY: run-bios
run-bios: $(IMAGE_NAME).iso
	qemu-system-x86_64 -M q35 -m 2G -cdrom $(IMAGE_NAME).iso -boot d

.PHONY: run-hdd-bios
run-hdd-bios: $(IMAGE_NAME).hdd
	qemu-system-x86_64 -M q35 -m 2G -hda $(IMAGE_NAME).hdd

.PHONY: ovmf
ovmf: ovmf-$(ARCH)

ovmf-x86_64:
	mkdir -p ovmf-x86_64
	cd ovmf-x86_64 && curl -o OVMF.fd https://retrage.github.io/edk2-nightly/bin/RELEASEX64_OVMF.fd

ovmf-aarch64:
	mkdir -p ovmf-aarch64
	cd ovmf-aarch64 && curl -o OVMF.fd https://retrage.github.io/edk2-nightly/bin/RELEASEAARCH64_QEMU_EFI.fd

ovmf-riscv64:
	mkdir -p ovmf-riscv64
	cd ovmf-riscv64 && curl -o OVMF.fd https://retrage.github.io/edk2-nightly/bin/RELEASERISCV64_VIRT.fd && dd if=/dev/zero of=OVMF.fd bs=1 count=0 seek=33554432

limine:
	git clone https://github.com/limine-bootloader/limine.git --branch=v5.x-branch-binary --depth=1
	$(MAKE) -C limine CC="$(HOST_CC)"

.PHONY: kernel
kernel:
	$(MAKE) -C kernel

$(IMAGE_NAME).iso: limine kernel
	rm -rf iso_root
	mkdir -p iso_root
	cp kernel/kernel.elf limine.cfg iso_root/
	mkdir -p iso_root/EFI/BOOT
ifeq ($(ARCH),x86_64)
	cp -v limine/limine-bios.sys limine/limine-bios-cd.bin limine/limine-uefi-cd.bin iso_root/
	cp -v limine/BOOTX64.EFI iso_root/EFI/BOOT/
	cp -v limine/BOOTIA32.EFI iso_root/EFI/BOOT/
	xorriso -as mkisofs -b limine-bios-cd.bin \
		-no-emul-boot -boot-load-size 4 -boot-info-table \
		--efi-boot limine-uefi-cd.bin \
		-efi-boot-part --efi-boot-image --protective-msdos-label \
		iso_root -o $(IMAGE_NAME).iso
	./limine/limine bios-install $(IMAGE_NAME).iso
else ifeq ($(ARCH),aarch64)
	cp -v limine/limine-uefi-cd.bin iso_root/
	cp -v limine/BOOTAA64.EFI iso_root/EFI/BOOT/
	xorriso -as mkisofs \
		--efi-boot limine-uefi-cd.bin \
		-efi-boot-part --efi-boot-image --protective-msdos-label \
		iso_root -o $(IMAGE_NAME).iso
else ifeq ($(ARCH),riscv64)
	cp -v limine/limine-uefi-cd.bin iso_root/
	cp -v limine/BOOTRISCV64.EFI iso_root/EFI/BOOT/
	xorriso -as mkisofs \
		--efi-boot limine-uefi-cd.bin \
		-efi-boot-part --efi-boot-image --protective-msdos-label \
		iso_root -o $(IMAGE_NAME).iso
endif
	rm -rf iso_root

$(IMAGE_NAME).hdd: limine kernel
	rm -f $(IMAGE_NAME).hdd
	dd if=/dev/zero bs=1M count=0 seek=64 of=$(IMAGE_NAME).hdd
	parted -s $(IMAGE_NAME).hdd mklabel gpt
	parted -s $(IMAGE_NAME).hdd mkpart ESP fat32 2048s 100%
	parted -s $(IMAGE_NAME).hdd set 1 esp on
ifeq ($(ARCH),x86_64)
	./limine/limine bios-install $(IMAGE_NAME).hdd
endif
	sudo losetup -Pf --show $(IMAGE_NAME).hdd >loopback_dev
	sudo mkfs.fat -F 32 `cat loopback_dev`p1
	mkdir -p img_mount
	sudo mount `cat loopback_dev`p1 img_mount
	sudo mkdir -p img_mount/EFI/BOOT
	sudo cp -v kernel/kernel.elf limine.cfg img_mount/
ifeq ($(ARCH),x86_64)
	sudo cp -v limine/limine-bios.sys img_mount/
	sudo cp -v limine/BOOTX64.EFI img_mount/EFI/BOOT/
	sudo cp -v limine/BOOTIA32.EFI img_mount/EFI/BOOT/
else ifeq ($(ARCH),aarch64)
	sudo cp -v limine/BOOTAA64.EFI img_mount/EFI/BOOT/
else ifeq ($(ARCH),riscv64)
	sudo cp -v limine/BOOTRISCV64.EFI img_mount/EFI/BOOT/
endif
	sync
	sudo umount img_mount
	sudo losetup -d `cat loopback_dev`
	rm -rf loopback_dev img_mount

.PHONY: clean
clean:
	rm -rf iso_root $(IMAGE_NAME).iso $(IMAGE_NAME).hdd
	$(MAKE) -C kernel clean

.PHONY: distclean
distclean: clean
	rm -rf limine ovmf*
	$(MAKE) -C kernel distclean
