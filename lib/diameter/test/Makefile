# 
# %CopyrightBegin%
#
# Copyright Ericsson AB 2010-2011. All Rights Reserved.
#
# The contents of this file are subject to the Erlang Public License,
# Version 1.1, (the "License"); you may not use this file except in
# compliance with the License. You should have received a copy of the
# Erlang Public License along with this software. If not, it can be
# retrieved online at http://www.erlang.org/.
#
# Software distributed under the License is distributed on an "AS IS"
# basis, WITHOUT WARRANTY OF ANY KIND, either express or implied. See
# the License for the specific language governing rights and limitations
# under the License.
#
# %CopyrightEnd%

ifeq ($(ERL_TOP),)
include $(DIAMETER_TOP)/make/target.mk
include $(DIAMETER_TOP)/make/$(TARGET)/rules.mk
else
include $(ERL_TOP)/make/target.mk
include $(ERL_TOP)/make/$(TARGET)/otp.mk
endif

# ----------------------------------------------------
# Application version
# ----------------------------------------------------

include ../vsn.mk

VSN = $(DIAMETER_VSN)

# ----------------------------------------------------
# Release directory specification
# ----------------------------------------------------

RELSYSDIR = $(RELEASE_PATH)/diameter_test

# ----------------------------------------------------
# Target Specs
# ----------------------------------------------------

include modules.mk

ERL_FILES    = $(MODULES:%=%.erl)
TARGET_FILES = $(MODULES:%=%.$(EMULATOR))

SUITE_MODULES = $(filter diameter_%_SUITE, $(MODULES))
SUITES        = $(SUITE_MODULES:diameter_%_SUITE=%)

DATA_DIRS = $(sort $(dir $(DATA)))

# ----------------------------------------------------
# FLAGS
# ----------------------------------------------------

# This is only used to compile suite locally when running with a
# target like 'all' below. Target release_tests only installs source.
ERL_COMPILE_FLAGS += +warn_export_vars \
                     +warn_unused_vars \
                     -I ../include \
                     -I ../src/gen

# ----------------------------------------------------
# Targets
# ----------------------------------------------------

all: opt

run: $(SUITES)

debug opt: $(TARGET_FILES)

clean:
	rm -f $(TARGET_FILES) 
	rm -f depend.mk

realclean: clean
	rm -rf log

docs:

list = echo $(1):; echo $($(1)) | tr ' ' '\n' | sort | sed 's@^@  @'

info:
	@echo ========================================
	@$(call list,MODULES)
	@echo
	@$(call list,HRL_FILES)
	@echo
	@$(call list,SUITES)
	@echo ========================================

help:
	@echo ========================================
	@echo "Useful targets:"
	@echo
	@echo "    all:"
	@echo "        Compile all test suites."
	@echo
	@echo "    run:"
	@echo "        Compile and run all test suites."
	@echo
	@echo "    $(SUITES):"
	@echo "        Compile and run a specific test suite."
	@echo
	@echo "    clean | realclean:"
	@echo "        Remove generated files."
	@echo
	@echo "    info:"
	@echo "        Echo some relevant variables."
	@echo ========================================

.PHONY: all run clean debug docs help info opt realclean

# ----------------------------------------------------
# Special Targets
# ----------------------------------------------------

# Exit with a non-zero status if the output looks to indicate failure.
# diameter_ct:run/1 itself can't tell (it seems). The absolute -pa is
# because ct will change directories.
$(SUITES): log opt
	$(ERL) -noinput \
	       -pa $(realpath ../ebin) \
	       -sname diameter_test_$@ \
	       -s diameter_ct run diameter_$@_SUITE \
	       -s init stop \
	| awk '{print} / FAILED /{rc=1} END{exit rc}' rc=0
# Shorter in sed but requires a GNU extension (ie. Q).

log:
	mkdir $@

.PHONY: $(SUITES)

# ----------------------------------------------------
# Release Targets
# ---------------------------------------------------- 

/%: % force
	sed -f release.sed $< > $(RELSYSDIR)$@

ifeq ($(ERL_TOP),)
include $(DIAMETER_TOP)/make/release_targets.mk
else
include $(ERL_TOP)/make/otp_release_targets.mk
endif

release_spec release_docs_spec:

release_tests_spec:
	$(INSTALL_DIR)  $(RELSYSDIR)
	$(INSTALL_DATA) $(TEST_SPEC_FILE) \
	                $(COVER_SPEC_FILE) \
	                $(HRL_FILES) \
	                $(RELSYSDIR)
	$(MAKE) $(DATA_DIRS:%/=release_data_%)
	$(MAKE) $(ERL_FILES:%=/%)

$(DATA_DIRS:%/=release_data_%): release_data_%:
	$(INSTALL_DIR) $(RELSYSDIR)/$*
	$(INSTALL_DATA) $(filter $*/%, $(DATA)) $(RELSYSDIR)/$*

force:

.PHONY: release_spec release_docs_spec release_test_specs
.PHONY: force
.PHONY: $(DATA_DIRS:%/=release_data_%)

# Can't just make $(ERL_FILES:%=/%) phony since then implicit rule
# searching is skipped.

# ----------------------------------------------------

depend: depend.mk

# Generate dependencies makefile.
depend.mk: depend.sed $(MODULES:%=%.erl) Makefile
	(for f in $(MODULES); do \
	     (echo $$f; cat $$f.erl) | sed -f $<; \
	 done) \
	> $@

-include depend.mk

.PHONY: depend
