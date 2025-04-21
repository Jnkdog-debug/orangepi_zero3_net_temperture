KERNEL_DIR := /home/xyz/orangepi/orangepi-build/kernel/orange-pi-6.1-sun50iw9

ARCH := arm64
CROSS_COMPILE := aarch64-none-linux-gnu-
export ARCH CROSS_COMPILE

obj-m := dht11_driver.o
BUILD_DIR := build

all:
	$(MAKE) -C $(KERNEL_DIR) M=$(CURDIR) modules
	@mkdir -p $(BUILD_DIR)
	@mv -f *.o *.ko *.mod.c *.mod.o *.symvers *.order .*.*.cmd *.mod $(BUILD_DIR) 2>/dev/null || true

.PHONY: clean copy

clean:
	$(MAKE) -C $(KERNEL_DIR) M=$(CURDIR) clean
	@rm -rf $(BUILD_DIR)

copy:
	@sudo cp $(BUILD_DIR)/dht11_driver.ko /home/xyz/xyzsnfs
