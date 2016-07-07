SH     = $(shell which sh)
PWD    = $(shell pwd)

# Varan
VX     = $(shell which vx)

# Compilers
GCC    = $(shell which gcc)
GCC_LD_LIBRARY_PATH =
CLANG  = $(shell which clang)

# For debugging
STRIP  = $(shell which true)
CFLAGS = -O0 -g

# Where each version expects to be installed
INSTALL_DIR       = $(PWD)/install
INSTALL_BUILD_DIR = $(INSTALL_DIR)/memcached

# Where different versions are installed
INSTALL_GCC        = $(INSTALL_DIR)/gcc-system
INSTALL_GCC_ASAN   = $(INSTALL_DIR)/gcc-asan-system
INSTALL_CLANG      = $(INSTALL_DIR)/clang-system
INSTALL_CLANG_ASAN = $(INSTALL_DIR)/clang-asan-system

# Binaries inside each build directory
MEMCACHED_GCC        = $(INSTALL_GCC)/bin/memcached
MEMCACHED_GCC_ASAN   = $(INSTALL_GCC_ASAN)/bin/memcached
MEMCACHED_CLANG      = $(INSTALL_CLANG)/bin/memcached
MEMCACHED_CLANG_ASAN = $(INSTALL_CLANG_ASAN)/bin/memcached

# Script to run MEMCACHED with Varan
MEMCACHED_DIR = $(PWD)/bin
MEMCACHED     = $(MEMCACHED_DIR)/memcached

# Command lines to run MEMCACHED with Varan inside script
RUN_GCC_GCC          = LD_LIBRARY_PATH="$(GCC_LD_LIBRARY_PATH)"                             $(VX) $(MEMCACHED_GCC) $(MEMCACHED_GCC)          -- \$$@
RUN_GCC_GCC_ASAN     = LD_LIBRARY_PATH="$(GCC_LD_LIBRARY_PATH)" ASAN_OPTIONS="handle_segv=0" $(VX) $(MEMCACHED_GCC) $(MEMCACHED_GCC_ASAN)     -- \$$@
RUN_CLANG_CLANG      =                              $(VX) $(MEMCACHED_CLANG) $(MEMCACHED_CLANG)      -- \$$@
RUN_CLANG_CLANG_ASAN = ASAN_OPTIONS="handle_segv=0" $(VX) $(MEMCACHED_CLANG) $(MEMCACHED_CLANG_ASAN) -- \$$@

# Main target
all: vx-gcc-gcc_asan

# ==============================================================================
# GCC targets
# ==============================================================================

# Set up script for native leader and native follower built with gcc
vx-gcc-gcc: $(MEMCACHED_GCC)
	rm -rf $(INSTALL_BUILD_DIR) || true
	cp -r $(INSTALL_GCC) $(INSTALL_BUILD_DIR)
	rm -rf $(MEMCACHED_DIR) || true
	mkdir -p $(MEMCACHED_DIR)
	echo "#!$(SH)"        >  $(MEMCACHED)
	echo "$(RUN_GCC_GCC)" >> $(MEMCACHED)
	chmod a+x $(MEMCACHED)

# Set up script for native leader and sanitized follower built with gcc
vx-gcc-gcc_asan: $(MEMCACHED_GCC) $(MEMCACHED_GCC_ASAN)
	rm -rf $(INSTALL_BUILD_DIR) || true
	cp -r $(INSTALL_GCC) $(INSTALL_BUILD_DIR)
	rm -rf $(MEMCACHED_DIR) || true
	mkdir -p $(MEMCACHED_DIR)
	echo "#!$(SH)"             >  $(MEMCACHED)
	echo "$(RUN_GCC_GCC_ASAN)" >> $(MEMCACHED)
	chmod a+x $(MEMCACHED)

# Build using gcc and no sanitization
$(MEMCACHED_GCC):
	rm -rf $(INSTALL_BUILD_DIR) || true
	make -C .. clean || true
	cd .. ; CC=$(GCC)                        \
  	CFLAGS="$(CFLAGS)"                       \
  	STRIP="$(STRIP)"                         \
	LD_LIBRARY_PATH="$(GCC_LD_LIBRARY_PATH)" \
  	./configure                              \
  	--prefix=$(INSTALL_BUILD_DIR)
	make -C .. V=1
	make -C .. install V=1
	rm -rf $(INSTALL_GCC) || true
	mv $(INSTALL_BUILD_DIR) $(INSTALL_GCC)

# Build using gcc and sanitization
$(MEMCACHED_GCC_ASAN):
	rm -rf $(INSTALL_BUILD_DIR) || true
	make -C .. clean || true
	cd .. ; CC=$(GCC)                        \
  	CFLAGS="$(CFLAGS) -fsanitize=address"    \
  	LDFLAGS="-fsanitize=address"             \
  	STRIP="$(STRIP)"                         \
	LD_LIBRARY_PATH="$(GCC_LD_LIBRARY_PATH)" \
  	./configure                              \
  	--prefix=$(INSTALL_BUILD_DIR)
	make -C ..
	make -C .. install
	rm -rf $(INSTALL_GCC_ASAN) || true
	mv $(INSTALL_BUILD_DIR) $(INSTALL_GCC_ASAN)


# ==============================================================================
# misc targets
# ==============================================================================

clean:
	make -C .. clean || true
	rm -rf $(INSTALL_DIR) || true
	rm -rf $(MEMCACHED_DIR) || true
