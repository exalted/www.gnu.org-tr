# This is -*-makefile-gmake-*-, because we adore GNU make.
# Copyright (C) 2008, 2009 Free Software Foundation, Inc.

# This file is part of GNUnited Nations.

# GNUnited Nations is free software: you can redistribute it and/or
# modify it under the terms of the GNU General Public License as
# published by the Free Software Foundation, either version 3 of the
# License, or (at your option) any later version.

# GNUnited Nations is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.

# You should have received a copy of the GNU General Public License
# along with GNUnited Nations.  If not, see <http://www.gnu.org/licenses/>.

########################################################################
### TRANSLATORS: Rename this file as GNUmakefile and install it in the #
### root of your project's *Sources* repository.  For details, see the #
### section "PO Files and Team" in the manual.                         #
########################################################################

### DEPENDENCIES ###
# GNU make >= 3.81 (prereleases are OK too)
# GNU gettext >= 0.16
# CVS
# Subversion (if the www-LANG repository is SVN)
# GNU Bzr (if the www-LANG repository is Bzr)
# Mercurial (if the www-LANG repository is Hg)
# GNU Arch (if the www-LANG repository is Arch)

SHELL = /bin/bash

# Set this variable to your language code.
TEAM := tr

# The relative or absolute path to the working copy of the master
# "www" repository; must end with a trailing slash.
wwwdir := ../../www/webpages/

# Adjust these variables if you don't have the programs in your PATH.
MSGMERGE := msgmerge
MSGFMT := msgfmt
CVS := cvs
DIFF := diff
SVN := svn
BZR := bzr
HG  := hg
# Baz can be used alternatively; its commands are compatible.
TLA := tla

translations := $(shell find -name '*.$(TEAM).po' | sort)
log := "GNUN: \"Güncellenen özgün metin ile otomatik olarak birleştirildi.\""

# Determine the VCS.
REPO := $(shell (test -d CVS && echo CVS) || (test -d .svn && echo SVN) \
	  || (test -d .bzr && echo Bzr) || (test -d .hg && echo Hg) \
	  || (test -d \{arch\} && echo Arch))
ifndef REPO
$(error Unsupported Version Control System)
endif

# For those who love details.
ifdef VERBOSE
$(info Repository: $(REPO))
$(info translations = $(translations))
MSGMERGEVERBOSE := --verbose
ECHO := echo $$file: ;
CVSQUIET :=
DIFFQUIET := --unified --unidirectional-new-file --report-identical-files
# Also applicable for Hg.
BZRQUIET := --verbose
else
CVSQUIET := -q
DIFFQUIET ?= --brief
BZRQUIET := --quiet
endif

# The command to update the CVS repositories.
define cvs-update
$(CVS) $(CVSQUIET) update -d -P
endef

# The command to update the Subversion repository.
define svn-update
$(SVN) $(CVSQUIET) update
endef

.PHONY: all
all: update sync

# Update the master and the team repositories.
.PHONY: update
update:
ifeq ($(VCS),yes)
	@echo Updating the repositories...
	cd $(wwwdir) && $(cvs-update)
ifeq ($(REPO),CVS)
	$(cvs-update)
else ifeq ($(REPO),SVN)
	$(svn-update)
else ifeq ($(REPO),Bzr)
	$(BZR) pull $(BZRQUIET)
else ifeq ($(REPO),Hg)
# The "fetch" extension is not guaranteed to be available, and/or
# enabled in user's ~/.hgrc.
	$(HG) pull --update $(BZRQUIET)
else ifeq ($(REPO),Arch)
	$(TLA) update
endif
else
	$(info Repositories were not updated, you might want "make VCS=yes".)
endif

# Synchronize (update) the PO files from the master POTs.
.PHONY: sync
sync: update
	@for file in $(translations) ; do \
	  $(ECHO) $(MSGMERGE) $(MSGMERGEVERBOSE) --quiet --update --previous \
	    $$file \
	    $(wwwdir)`dirname $$file`/po/`basename $${file/.$(TEAM).po/.pot}`; \
	done
ifeq ($(VCS),yes)
ifeq ($(REPO),CVS)
	$(CVS) commit -m $(log)
else ifeq ($(REPO),SVN)
	$(SVN) commit -m $(log)
else ifeq ($(REPO),Bzr)
# The behavior of `bzr commit' is not very script-friendly: it will
# exit with an error if there are no changes to commit.
	if $(BZR) status --versioned --short | grep --quiet '^ M'; then \
	  $(BZR) commit $(BZRQUIET) -m $(log) && $(BZR) push $(BZRQUIET); \
	else \
	  true; \
	fi
else ifeq ($(REPO),Hg)
	$(HG) commit $(BZRQUIET) -m $(log) && $(HG) push $(BZRQUIET)
else ifeq ($(REPO),Arch)
# Arch is so dumb that it will do a bogus commit (adding another
# absolutely useless revision) even if there are no changes.
# Fortunately, the exit status of `tla changes' is sane.
	$(TLA) changes >/dev/null || $(TLA) commit -s $(log)
endif
endif

# Compare (diff) your working copy with the master repository
.PHONY: diff
diff:
	@for file in $(translations) ; do \
	  $(DIFF) $(DIFFQUIET) \
	    $(wwwdir)`dirname $$file`/po/`basename $${file}` $$file ; \
	done

# Helper target to check which articles have to be updated.
.PHONY: report
report:
	@for file in $(translations) ; do \
	  LC_ALL=C $(MSGFMT) --statistics -o /dev/null $$file 2>&1 \
	    | egrep '(fuzzy|untranslated)' \
	      && echo -e "$${file#./} needs updating.\n" || true ; \
	done

# Helper target to check which articles are complete. (Reverse report)
.PHONY: rreport
rreport:
	@for file in $(translations) ; do \
	  LC_ALL=C $(MSGFMT) --statistics -o /dev/null $$file 2>&1 \
	    | egrep -v '(fuzzy|untranslated)' \
	      && echo -e "$${file#./} is complete.\n" || true ; \
	done
