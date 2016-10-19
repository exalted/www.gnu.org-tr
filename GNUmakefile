# This is -*-makefile-gmake-*-, because we adore GNU make.
# Copyright (C) 2008, 2009, 2010, 2011, 2012,
#   2014, 2016 Free Software Foundation, Inc.

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
# along with GNUnited Nations.  If not, see <https://www.gnu.org/licenses/>.

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
# Git (if the www-LANG repository is Git)
# Mercurial (if the www-LANG repository is Hg)
# GNU Arch (if the www-LANG repository is Arch)

SHELL = /bin/bash

# Set this variable to your language code.
TEAM := tr
ifndef TEAM
TEAM := $(shell find . -name \*.po | head -n 1 | sed 's/\.po$$//; s/.*\.//')
endif

# The relative or absolute path to the working copy of the master
# "www" repository; must end with a trailing slash.
wwwdir := ../www-web/

www_gnun_dir := $(wwwdir)server/gnun/

# Adjust these variables if you don't have the programs in your PATH.
MSGMERGE := /usr/local/Cellar/gettext/0.19.8.1/bin/msgmerge
MSGMERGEFLAGS := --previous --backup=none
MSGFMT := /usr/local/Cellar/gettext/0.19.8.1/bin/msgfmt
MSGCAT := /usr/local/Cellar/gettext/0.19.8.1/bin/msgcat
MSGATTRIB := /usr/local/Cellar/gettext/0.19.8.1/bin/msgattrib
CVS := cvs
SVN := svn
BZR := bzr
GIT := git
HG  := hg
# Baz can be used alternatively; its commands are compatible.
TLA := tla
# Default period of notifications.
NOTIFICATION_PERIOD := 7
# URL specifications; used in notifications to generate URLs of items.
# Root URL for "www" files.
WWW_URL := https://www.gnu.org/
# Prefix and postfix of URLs of team's files.
TEAM_URL_PREFIX := http://cvs.savannah.gnu.org/viewvc/*checkout*/www-bg/
TEAM_URL_POSTFIX := ?root=www-bg
# The program to generate differences of two versions of a PO file.
# Those files will be sent with notifications as attachments.
ifndef DIFF_PO
DIFF_PO := $(shell which gnun-diff-po)
# Detect a UTF-8 locale (msgexec doesn't like processing UTF-8-encoded POs
# in an incompatible locale).
ifneq (,$(DIFF_PO))
DIFF_PO_LANG := $(shell \
  locale \
  | if locale | egrep "^LC_ALL=." > /dev/null; then \
      grep "^LC_ALL"; \
    else \
      cat; \
    fi \
  | egrep -i "=.*utf-?8" > /dev/null \
  || locale -a | sed 's/^/LC_ALL=/;/en_US\.utf-?8/!d;q' | egrep "." \
  || locale -a | sed 's/^/LC_ALL=/;/\.utf-?8/!d;q';)
DIFF_PO := $(DIFF_PO_LANG) $(DIFF_PO)
endif
endif
# The program to add differences against previous msgids
# to fuzzy translations (and remove those differences from up-to-date
# translations).
ifndef ADD_FUZZY_DIFF
ADD_FUZZY_DIFF := $(shell which gnun-add-fuzzy-diff)
endif
# The program to send notifications.
ifndef GNUN_MAIL
ifneq (,$(DIFF_PO))
# Use mutt since mail doesn't accept attachments.
GNUN_MAIL := mutt
else # eq (,$(DIFF_PO))
GNUN_MAIL := mail
endif # neq (,$(DIFF_PO))
endif # ndef GNUN_MAIL
# Invoke with NOTIFY=yes to actually enable notifications
ifneq (yes,$(NOTIFY))
GNUN_MAIL := { echo $(GNUN_MAIL)
MAIL_TAIL := cat; };
else
MAIL_TAIL :=
endif

translations := $(shell find . -name '*.$(TEAM).po' \
                             ! -path ./server/gnun/\* | sort)
# Master compendium (if present)
master := $(wildcard $(wwwdir)server/gnun/compendia/master.$(TEAM).po)

log := "Automatic merge from the master repository."
# Warning message for the `publish' rule.
pubwmsg := "Warning (%s): \`%s\' does not exist\
\n    (either obsolete or \`cvsupdate\' in $(wwwdir) needed).\n"

# Determine the VCS.
REPO := $(shell (test -d CVS && echo CVS) || (test -d .svn && echo SVN) \
	  || (test -d .bzr && echo Bzr) || (test -d .git && echo Git) \
	  || (test -d .hg && echo Hg) || (test -d \{arch\} && echo Arch))
ifndef REPO
$(error Unsupported Version Control System)
endif

# For those who love details.
ifdef VERBOSE
$(info Repository: $(REPO))
$(info translations = $(translations))
MSGMERGEVERBOSE := --verbose
CVSQUIET :=
# Applicable for Bzr, Git and Hg.
QUIET := --verbose
else
CVSQUIET := -q
QUIET := --quiet
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
all:
	$(MAKE) $(MAKEFLAGS) sync && $(MAKE) $(MAKEFLAGS) format \
&& $(MAKE) $(MAKEFLAGS) notify

# Update the master and the team repositories.
.PHONY: update update-team update-www
update: update-team update-www
update-www:
	@echo Updating the repositories...
	cd $(wwwdir) && $(cvs-update)
update-team:
ifeq ($(REPO),CVS)
	$(cvs-update)
else ifeq ($(REPO),SVN)
	$(svn-update)
else ifeq ($(REPO),Bzr)
	$(BZR) pull $(QUIET)
else ifeq ($(REPO),Git)
	$(GIT) pull $(QUIET)
else ifeq ($(REPO),Hg)
# The "fetch" extension is not guaranteed to be available, and/or
# enabled in user's ~/.hgrc.
	$(HG) pull --update $(QUIET)
else ifeq ($(REPO),Arch)
	$(TLA) update
endif

# Synchronize (update) the PO files from the master POTs.
# The revision of the PO file from ${wwwdir} is used as a possible
# source for missing translations, which covers the case when the
# coordinator updates the translation directly in `www' repository.
.PHONY: sync
# Actual synchoronizations are defined as dependencies
# to enable parallel processing.
sync: update
ifeq ($(VCS),yes)
ifeq ($(REPO),CVS)
	$(CVS) commit -m $(log)
else ifeq ($(REPO),SVN)
	$(SVN) commit -m $(log)
else ifeq ($(REPO),Bzr)
# The behavior of `bzr commit' is not very script-friendly: it will
# exit with an error if there are no changes to commit.
	if $(BZR) status --versioned --short | grep '^ M' > /dev/null; then \
	  $(BZR) commit $(QUIET) -m $(log) && $(BZR) push $(QUIET); \
	else \
	  true; \
	fi
else ifeq ($(REPO),Git)
# Git (`git commit', to be precise) will exit with an error if there
# are only untracked files present (a common situation).  Sadly, there
# doesn't seem to be a decent workaround, so exit status is ignored.
	-$(GIT) commit --all $(QUIET) -m $(log)
	$(GIT) push $(QUIET)
else ifeq ($(REPO),Hg)
	$(HG) commit $(QUIET) -m $(log) && $(HG) push $(QUIET)
else ifeq ($(REPO),Arch)
# Arch is so dumb that it will do a bogus commit (adding another
# absolutely useless revision) even if there are no changes.
# Fortunately, the exit status of `tla changes' is sane.
	$(TLA) changes >/dev/null || $(TLA) commit -s $(log)
endif
endif

sync-master :=
# Sync master compendium when present.
ifneq (,$(master))
team-master := $(wildcard server/gnun/compendia/master.$(TEAM).po)
ifneq (,$(team-master))
.PHONY: sync-master
sync-master:
	@if $(call cmp-POs,$(master),$(team-master)); then \
	  echo "$(team-master): Already in sync."; \
	else \
	  echo "$(team-master): Copying from \`www'."; \
	  cp $(master) $(team-master); \
	fi
	@$(call echo-statistics,$(team-master))
sync: sync-master
sync-master := sync-master
endif
endif

# The command to compare PO files; "extracted" comments (including
# `# type: ...'), old messages, dates are considered insignificant.
define cmp-POs
{ $(MSGATTRIB) --no-obsolete --force-po -w 79 -o $1.tmp.po $1; \
  $(MSGATTRIB) --no-obsolete --force-po -w 79 $2 \
  | diff $1.tmp.po - | grep '^[<>] ' | egrep -v \
'^..($$|#\. |# type: |"(POT-Creation-Date|PO-Revision-Date|(X-)?Outdated-Since):)' \
   > /dev/null; \
  status=$$?; rm $1.tmp.po; test $$status != 0; }
endef

# Merge a file.
# When the master compendium is present, its translations override
# the translations from the PO file.
#
#     Input files
#
# $1: the PO file to merge.
# $$$$pot: Its POT.
# $$$$comp (when non-empty): an additional argument with PO file
# from `www' used as compendium, in order to copy new translations
# from `www' to `www-$(TEAM)'.
#
#     Intermediate files
#
# $1-tmp0.po: the set of msgids common to master.$(TEAM).po and $1.
# $1-tmp1.po: $1 without the set defined in $1-tmp0.po (when the latter
# is not empty).
# $$$$po0: the PO file to acutally merge ($1-tmp1.po when it exists,
# otherwise $1).
define merge-file
$(if $(master), \
  po0=$1; \
  $(MSGCAT) --more-than=1 --use-first $1 $(master) > $1-tmp0.po; \
  if test -s $1-tmp0.po; then \
    $(MSGCAT) --less-than=2 --use-first $1 $1-tmp0.po > $1-tmp1.po; \
  fi; \
  if test -s $1-tmp1.po; then  po0=$1-tmp1.po; fi; \
  $(MSGMERGE) --previous $(MSGMERGEVERBOSE) -C $(master) $$$$comp \
    -o $1 $$$$po0 $$$$pot 2>&1; \
  $(RM) $1-tmp0.po $1-tmp1.po \
, \
  $(MSGMERGE) $(MSGMERGEFLAGS) $(MSGMERGEVERBOSE) $$$$comp \
    --update $1 $$$$pot 2>&1 \
)
endef

# Output statistics for PO file $1.
define echo-statistics
  echo "   " `$(MSGFMT) -o /dev/null --statistics $1 2>&1`
endef

# Synchronize (merge) a file.
# The translations from $(master) override the translations from PO files
# ($(master) update triggers remerging all files);
# new translations from `www' propagate to `www-$(TEAM)'; current translations
# from `www' replace fuzzy translations in `www-$(TEAM)'.
define sync-file
.PNONY: sync-$(1)
sync-$(1): $(sync-master)
	@file=$(1); \
	  pot=$(wwwdir)`dirname $1`/po/`basename $$$${file%.$(TEAM).po}.pot`; \
	  test -f $$$$pot || pot=$$$$pot.opt; \
	  if test ! -f $$$$pot; then \
	    echo "Warning: $$$${file#./} has no equivalent .pot in www."; \
	  else \
	    www_po=$(wwwdir)`dirname $1`/po/`basename $1`; comp=; \
	    if test -f $$$${www_po}; then \
	      comp="-C $$$$www_po"; \
	      $$(if $(master), test $$$$file -nt $(master) && ) \
	      $$(call cmp-POs,$1,$$$${www_po}) \
	        && echo "$$$${file#./}: Already in sync." \
	        || { \
		     echo -n "$$$${file#./}: Merging"; \
		     $(MSGATTRIB) --no-fuzzy -o $(1)-tmp.www.po $$$$www_po  2>&1; \
		     $(MSGATTRIB) --fuzzy -o $(1)-tmp.po $1 2>&1; \
		     if test -s $(1)-tmp.po && test -s $(1)-tmp.www.po; then \
		       $(MSGCAT) --use-first --more-than=1 \
			  $(1)-tmp.www.po $(1)-tmp.po 2>&1 \
		       | $(MSGCAT) --use-first --less-than=2 -o $(1) - $(1); \
		     fi; \
		     $(merge-file); \
		     $(RM) $(1)-tmp.www.po $(1)-tmp.po; \
		   } \
	    else \
	      echo -n "$$$${file#./}: Merging new translation"; \
	      $(merge-file); \
	    fi; \
	    $(if $(ADD_FUZZY_DIFF), $(ADD_FUZZY_DIFF) $1 > $1.tmp \
		 && cmp -s $1 $1.tmp || cp $1.tmp $1; $(RM) $1.tmp;) \
	    $$(call echo-statistics,$1); \
	  fi
sync: sync-$(1)
endef
$(foreach file, $(patsubst ./%, %, $(translations)), \
                  $(eval $(call sync-file,$(file))))

# Import translated file lists from www.
-include $(www_gnun_dir)gnun.mk
# Assign priorities to translations for report.
-include $(www_gnun_dir)priorities.mk

define sorted-files
priority-articles-$(1) := \
  $(filter $(foreach article,${priority-articles},\
             ./${article}.${TEAM}.po), $(2))
important-articles-$(1) := \
  $(filter $(foreach article,${important-articles},\
             ./${article}.${TEAM}.po), $(2))
important-dir-$(1) := \
$$(filter-out $${priority-articles-$(1)} $${important-articles-$(1)}, \
  $$(filter $$(addsuffix /%, $(addprefix ./,${important-directories})), $(2)))
other-$(1) := \
  $$(filter-out $${priority-articles-$(1)} $${important-articles-$(1)} \
    $${important-dir-$(1)}, $(2))
endef

$(eval $(call sorted-files,pos,${translations}))

# Figure out what translatable articles live in www.
ifeq (,$(ALL_DIRS))
pots := $(shell find $(wwwdir) -name \*.pot ! -path \*/server/gnun/\*)
else # ! eq (,$(ALL_DIRS))
template-pots := $(addsuffix .pot, \
		   $(foreach template,$(extra-templates), \
		     $(dir $(addprefix $(wwwdir), \
		             $(template)))po/$(notdir $(template)))) \
                 $(addsuffix .pot.opt, \
		   $(foreach template,$(optional-templates), \
		     $(dir $(addprefix $(wwwdir), \
		             $(template)))po/$(notdir $(template))))
no-grace-items := $(no-grace-articles)
no-grace-pot := $(no-grace-items:%=%.pot)
articles := $(foreach dir,$(ALL_DIRS),$(addprefix $(dir)/po/,$(value $(dir))))
articles-pot := $(addprefix $(wwwdir),$(articles:%=%.pot))
root-articles := $(foreach root-article,$(ROOT), \
		   $(addprefix $(wwwdir)po/,$(root-article)))
root-articles-pot := $(root-articles:%=%.pot)
pots := $(articles-pot) $(root-articles-pot) $(template-pots)
endif # ! eq (,$(ALL_DIRS))

pos := \
$(wildcard $(patsubst %.pot,%.$(TEAM).po,$(filter %.pot,$(pots)))\
  $(patsubst %.pot.opt,%.$(TEAM).po,$(filter %.pot.opt,$(pots))))
team-pos := \
$(wildcard $(patsubst $(wwwdir)%,%,$(subst /po/,/,$(pos))))
htmls := \
$(wildcard $(subst /po/,/,\
             $(patsubst %.pot.opt,%.$(TEAM).html, $(filter %.pot.opt,$(pots)))\
             $(patsubst %.pot,%.$(TEAM).html, $(filter %.pot,$(pots)))))

# Team's translations that lack PO file.
# Note: optional templates can't have HTML translations, so $(filter %.pot...
html-only := \
$(strip \
  $(foreach pot, $(filter %.pot,$(pots)),\
    $(if $(findstring $(subst /po/,/,$(pot:%.pot=%.$(TEAM).html)),$(htmls)),\
      $(if $(or $(findstring $(pot:%.pot=%.$(TEAM).po),$(pos)),\
             $(findstring $(subst /po/,/,$(pot)),\
               $(addprefix $(wwwdir),$(team-pos)))\
            ),,\
        $(patsubst $(wwwdir)%.pot,%.$(TEAM).po,$(subst /po/,/,$(pot)))))\
   )\
 )
$(eval $(call sorted-files,html,${html-only}))

# Team's translations that lack PO file.
www-only := \
$(strip \
  $(foreach base, $(patsubst %.pot.opt,%,$(filter %.pot.opt,$(pots)))\
                  $(patsubst %.pot,%,$(filter %.pot,$(pots))),\
    $(if $(findstring $(base).$(TEAM).po,$(pos)),\
      $(if $(findstring $(subst /po/,/,$(base)).$(TEAM).po,\
             $(addprefix $(wwwdir),$(team-pos))),,\
        $(patsubst $(wwwdir)%,%.$(TEAM).po,$(subst /po/,/,$(base))))\
     )\
   )\
 )
$(eval $(call sorted-files,www,${www-only}))

# Function to report a group of PO files.
define report-pos
@$(if $(strip $($(1)-pos)$($(1)-html)$($(1)-www)), \
  $(if $(2), echo "  "$(strip $(2)); echo;) \
  $(if $($(1)-pos), \
    for file in $($(1)-pos); do \
      statistics=`LC_ALL=C $(MSGFMT) --statistics -o /dev/null $$file 2>&1 \
                  | egrep '(fuzzy|untranslated)'`; \
      if test -n "$$statistics"; then \
        echo "$${file#./}:" $$statistics; \
      fi; \
      www_po=$(wwwdir)`dirname $$file`/po/`basename $$file`; \
      pot=$${www_po%.$(TEAM).po}.pot; test -f $$pot || pot=$$pot.opt; \
      if ! test -f $$pot; then \
        echo "$${file#./}: no POT in \`www'."; \
	continue; \
      fi; \
      if test -f $$www_po; then \
	if $(call cmp-POs,$$www_po,$$file); then \
	  $(RM) $$file-diff.html; \
	else \
          www_statistics=`LC_ALL=C $(MSGFMT) --statistics -o /dev/null \
					$$www_po 2>&1 \
			  | egrep '(fuzzy|untranslated)'`; \
          case "@$$www_statistics@$$statistics@" in \
	    @?*@@ $(paren) \
	       echo \
"$${file#./}: the team version seems ready to post."; \
	       ;; \
	    * $(paren) \
	       echo \
"$${file#./}: \`www' and \`www-$(TEAM)' revisions are not consistent."; \
	       ;; \
          esac; \
	  $(if $(DIFF_PO), $(DIFF_PO) \
  --title "$${file#./}: www vs. www-$(TEAM) repository" \
  $$www_po $$file > $$file-diff.html;) \
	fi; \
      else \
        if test ".$$statistics" = .;then \
	  echo \
	    "$${file#./}: new translation seems ready to post."; \
	fi; \
      fi; \
    done;) \
  $(if $($(1)-www), \
    for file in $($(1)-www); do \
      echo "$${file#./}: present in \`www'$(comma) absent in \`www-$(TEAM)'."; \
    done;) \
  $(if $($(1)-html), \
    for file in $($(1)-html); do \
      echo \
        "$${file#./}: HTML-only translation$(comma) needs conversion to PO."; \
    done;) \
  $(if $(2), echo;))
endef

# Helper target to check which articles have to be updated.
.PHONY: report
report:
ifeq (,${priority-articles}${important-articles}${important-directories})
	$(call report-pos,other)
else #!eq (,${priority-articles}${important-articles}${important-directories})
	$(call report-pos,priority-articles,Priority Articles)
	$(call report-pos,important-articles,Important Articles)
	$(call report-pos,important-dir,\
          Other Articles from Important Directories)
	$(call report-pos,other,Other Translations)
endif #!eq (,${priority-articles}${important-articles}${important-directories})

report.txt: $(translations) $(wildcard priorities.mk)
	$(MAKE) report | grep -v '^make\[' > $@

# Helper target to rewrap all PO files; avoids spurious diffs when
# they get remerged by the official build.
# Formatting is defined per-file as dependencies of the main target
# to enable parallel processing.
.PHONY: format
define format-file
.PNONY: format-$(1)
format-$(1):
	@file=$(1); $(MSGCAT) -o $$$$file-tmp $$$$file; \
	  cmp -s $$$$file-tmp $$$$file || cp $$$$file-tmp $$$$file; \
	  $(RM) $$$$file-tmp; echo "  $$$${file#./} formatted."
format: format-$(1)
endef
$(foreach file, $(patsubst ./%, %, $(translations)), \
                  $(eval $(call format-file,$(file))))

# Helper target to copy all (supposedly) modified files to the `www'
# master repository.  A warning is printed if the corresponding
# directory in `www' cannot be found, or if the template is missing
# (which in almost all cases means that the original article has been
# renamed or deleted).
.PHONY: publish
publish: format
	@echo All edited .po files have been copied back to $(wwwdir)
define publish-file
.PNONY: publish-$(1)
publish-$(1):
	@file=$(1); wwwfdir=$(wwwdir)`dirname $$$$file`/po; \
	  pot=$$$${wwwfdir}/`basename $$$${file/.$(TEAM).po/.pot}.opt`; \
	  test -f $$$$pot || pot=$$$${pot%.opt}; \
	  wwwfile=$$$${wwwfdir}/`basename $$$$file`; \
	  if [ ! -d $$$$wwwfdir ]; then \
	    printf $(pubwmsg) "$$$${file#./}" "$$$$wwwfdir/"; \
	    exit 0; \
	  fi; \
	  if [ ! -f $$$$pot ]; then \
	    printf $(pubwmsg) "$$$${file#./}" "$$$$pot"; \
	    exit 0; \
	  fi; \
	  if [ $$$$file -nt $$$$wwwfile ]; then \
	    cp $$$$file $$$$wwwfile && echo "  $$$${file#./} published."; \
	  fi
publish: publish-$(1)
endef
$(foreach file, $(patsubst ./%, %, $(translations)), \
                  $(eval $(call publish-file,$(file))))

# Email aliases are defined through the file named `email-aliases'.
# Lines beginning with `#' are ignored.
# Every line should contain two or more colon-separated
# fields.  The first field is the identifier, the second
# field is space-separated list of email addresses,
# the third field is notification period in days
# ($(NOTIFICATION_PERIOD) by default): a notification
# is sent either when it changes, or when the period is over.
# The fourth field is comma-separated list of flags
# (`no-diffs' to disable sending PO differences in attachments,
#  `www' to report translations to be published).
# The lines without a colon are ignored.
HAVE-EMAIL-ALIASES := $(shell test -s email-aliases && echo yes)

# Per-translator notification rules are defined in the `nottab' file.
# The lines beginning with `#' are ignored.
# Every line should contain exactly two colon-separated fields:
# the first is an extended regular expression, the second is
# space-separated list of email aliases to notify about the files
# whose names match the expression.  The lines without a colon
# are ignored.
HAVE-NOTTAB := $(shell test -s nottab && echo yes)

.PHONY: notify
# The variables to use closing parentheses and commas in arguments
# of make functions.
paren := )
comma := ,
# The function to find options for the translator $(1);
# they are assigned to the shell variables `email', `period', `flags'.
define parse-email-aliases
$(if $(HAVE-EMAIL-ALIASES), \
  record=`grep -v '^#' email-aliases | grep '^$(subst .,\\.,$1):' \
  | head -n 1 | sed "s/^[^:]\+://"`; \
  email=$$$${record%%:*}; flags=; period=; \
  if test "x$$$$email" != "x$$$$record"; then \
    period=$$$${record#*:}; \
    case "x$$$$period" in \
	*:* $(paren) flags=$$$${period#*:}; period=$$$${period%%:*}; ;; \
    esac; \
  fi, echo "Note: no email alias for \`$1' found.")
endef
ifneq (,$(HAVE-NOTTAB))
# Extract identifiers of all translators.
translators := $(shell \
  sed '/^\#/d;/:/!d;s/^[^:]*://;s/:.*//;s/[[:space:]]\+/\n/g' nottab | sort -u)
define notify-translator
.PRECIOUS: $(1).note
.INTERMEDIATE: $(1).note.tmp
# The empty lines and lines beginning with a space are passed
# from report.txt to $(1).note.tmp unchanged.  Other lines begin
# with filenames (up to the first `:'); they come to $(1).note.tmp
# when and only when the filename matches one of regexps requested
# for that translator via nottab.
$(1).note.tmp: report.txt nottab $(wildcard email-aliases)
	@regex="`egrep -v '^#' nottab \
		 | egrep  ':(.+ )?$(subst .,\\.,$1)( |$$$$)' \
		 | while read line; do \
		     line=$$$${line//./\.}; \
		     echo -n "|($$$${line%%:*})"; \
		   done`"; \
	if test "x$$$$regex" = x; then \
	  regex=':(actually this must be impossible)'; \
	else \
	  regex="($$$${regex#|})"; \
	fi; \
	sed "s/^/@/" report.txt \
	| while read line; do \
	    line="$$$${line#@}"; \
	    if echo "$$$$line" | egrep "^( |$$$$)" > /dev/null; then \
	      echo "$$$$line"; continue; \
	    fi; \
	    file="$$$${line%%:*}"; \
	    if echo $$$$file | egrep "$$$$regex" > /dev/null; then \
	      echo "$$$$line"; \
	    fi; \
	  done > $(1).note.tmp
	@$(parse-email-aliases); \
case ",$$$$flags," in \
  *,www,* ) ;; \
  * ) \
    sed --in-place -e '/:.* seems ready to post\.$$$$/d' $(1).note.tmp \
    ;; \
esac

# Final notification file for the translator prepended with
# the timestamp of the latest notification.
$(1).note: $(1).note.tmp
	@if test -s $(1).note && sed "1d" < $(1).note \
	   | cmp -s - $(1).note.tmp &>/dev/null; then \
	  touch $(1).note; \
	else \
	  cp $(1).note.tmp $(1).note; \
	fi

.PHONY: inform-$(1)
# Extract the list of emails and period for the translator
# from email-aliases.  Send the notification to the list of emails
# when $(1).note is updated (that is, the timestamp is absent)
# or when the period has passed.  When sending, update
# the timestamp in $(1).note.
# The URLs of are added to the report for convenience because
# the notification is emailed (they would be pointless if it were
# displayed locally, therefore the files were readily available).
inform-$(1): $(1).note
ifneq (,$(HAVE-EMAIL-ALIASES))
	@$(parse-email-aliases); \
  case "x$$$$period" in \
    x ) period=$(NOTIFICATION_PERIOD); ;; \
  esac; \
  test "x$$$$email" != x \
    || { echo "Note: no email for \`$1' found in email-aliases."; exit 0; }; \
  period=$$$$(($$$$period*24*3600)); notify=yes; \
  case ",$$$$flags," in \
    *,force,* ) ;; \
    * ) \
      grep : $(1).note &> /dev/null \
      || { notify=no; echo "Note: No files to work on for \`$(1)'."; } \
      ;; \
  esac; \
  case $$$$notify in \
    yes ) \
      timestamp=`head -n1 $(1).note | grep '^#'`; \
      if test "x$$$$timestamp" != x; then \
        dt=$$$$((`date +%s` - $$$${timestamp#?})); \
        if test $$$$dt -lt $$$$period; then \
          notify=no; \
	  echo "Note: Elapsed time ($$$$dt) is less than \
period ($$$$period) for \`$(1)'.";\
        fi; \
      else \
       echo "Note: New notification for \`$(1)'."; \
      fi; \
      ;; \
  esac; \
  if test $$$$notify = yes; then \
    echo "Sending notification for \`$1' ($$$$email)..."; \
    case ",$$$$flags," in \
      *,no-diffs,* ) \
	echo "Note: attachments for \`$1' are disabled."; \
	attachments=; \
	;; \
      * ) \
	attachments=$(if $(DIFF_PO),`egrep \
'(revisions are not consistent|team version seems ready to post)\.$$$$'\
   $(1).note \
| sed 's/:.*//;s/$$$$/-diff.html/' \
| while read f; do if test -s $$$$f; then echo $$$$f; fi; done;`); \
	;; \
    esac; \
    if test "x$$$$attachments" != x; then \
      attachments="-a $$$$attachments --"; \
    fi; \
    grep -v '^#' $(1).note \
      $(if, Join subsequent lines reporting against the same file) \
      $(if, to avoid writing unnecessarily duplicated URLs.) \
      | sed " /:/ { :c; N; s/^\([^:]*:\)\(.*\)\n\1/\1\2@/; t c; h; \
                     s/\n[^\n]*$$$$//; p; g; s/[^\n]*\n//; /:/ b c }" \
      | sed "s/^/@/" \
      | while read line; do \
	  line="$$$${line#@}"; \
	  echo -n "$$$$line"; \
	  if echo "$$$$line" | grep "^[^ ]" > /dev/null ; then \
	    file=$$$${line%%:*}; name=$$$${file##*/}; dir=$$$${file%/*}/; \
	    if test ".$$$$dir" = ".$$$$file/"; then dir=""; fi; \
	    head="  ("; \
	    if test -f $$$$file; then \
	      echo; \
	      echo -n "$$$${head}$(TEAM_URL_PREFIX)$$$$file$(TEAM_URL_POSTFIX)"; \
	      head="   "; \
	    fi; \
	    if test -f $(wwwdir)$$$${dir}po/$$$$name; then \
	      echo; \
	      echo -n "$$$${head}$(WWW_URL)$$$${dir}po/$$$$name"; \
	      head="   "; \
	    fi; \
	    if echo "$$$$line" | grep -F "HTML-only translation" > /dev/null; then \
	      echo; \
	      echo -n "$$$${head}$(WWW_URL)$$$$dir$$$${name%po}html"; \
	      head="   "; \
	    fi; \
	    pot=$$$${name%.$(TEAM).po}.pot; \
	    if test -f $(wwwdir)$$$${dir}po/$$$$pot; then \
	      echo; \
	      echo -n "$$$${head}$(WWW_URL)$$$${dir}po/$$$$pot"; \
	    elif test -f $(wwwdir)$$$${dir}po/$$$$pot.opt; then \
	      echo; \
	      echo -n "$$$${head}$(WWW_URL)$$$${dir}po/$$$$pot.opt"; \
	    fi; \
	    echo ')'; \
	  fi; \
	  echo; \
	  done \
      $(if, "Unjoin" the lines reporting against the same file.) \
      | sed "/:.*@/{:cycle; s/^\([^:]*:\)\(.*\)@/\1\2\n\1/; t cycle;}" \
      | $(GNUN_MAIL) -s "Automatic Notification for \`"$1"'" \
			$$$$attachments $$$$email; $(MAIL_TAIL) \
    sed --in-place '1{/^#/d;}' $(1).note; \
    sed --in-place "1i#`date +%s`" $(1).note; \
  else \
    echo "Skipping notification for \`$1'..."; \
  fi
endif #ifneq (,$(HAVE-EMAIL-ALIASES))
notify: inform-$(1)
endef
$(foreach translator, $(translators), \
	  $(eval $(call notify-translator,$(translator))))
else # eq (,$(HAVE-NOTTAB))
notify: ; @echo Nobody to notify.
endif # eq (,$(HAVE-NOTTAB))

# Helper target to delete common auto-generated files.
.PHONY: clean
clean:
	@echo Deleting auto-generated files...
	@for file in $(translations); do \
	  $(RM) $$file~ $${file/.po/.mo} $$file.bak $$file-diff.html; \
	done
	@$(RM) report.txt *.note *.note.tmp
