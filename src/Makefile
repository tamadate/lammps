# LAMMPS multiple-machine -*- Makefile -*-

SHELL = /bin/bash
PYTHON = python
DYN_LIB = -ldl

#.IGNORE:

# Definitions

ROOT =   lmp
EXE =    lmp_$@
ARLIB =  liblammps_$@.a
SHLIB =  liblammps_$@.so
ARLINK = liblammps.a
SHLINK = liblammps.so
TMPNAME= tmp_$@_name
LMPLINK= -L. -llammps_$@

OBJDIR =   Obj_$@
OBJSHDIR = Obj_shared_$@

SRC =   $(wildcard *.cpp)
INC =   $(filter-out lmpinstalledpkgs.h lmpgitversion.h,$(wildcard *.h))
OBJ =   $(SRC:.cpp=.o)

SRCLIB = $(filter-out main.cpp,$(SRC))
OBJLIB = $(filter-out main.o,$(OBJ))

# Command-line options for mode: static (default), shared, or print

mode = static
objdir = $(OBJDIR)

ifeq ($(mode),shared)
objdir = $(OBJSHDIR)
endif


# Helper GNU make function for conversion to upper case without using shell commands
uppercase_TABLE:=a,A b,B c,C d,D e,E f,F g,G h,H i,I j,J k,K l,L m,M n,N o,O p,P q,Q r,R s,S t,T u,U v,V w,W x,X y,Y z,Z
uppercase_internal=$(if $1,$$(subst $(firstword $1),$(call uppercase_internal,$(wordlist 2,$(words $1),$1),$2)),$2)
uppercase=$(eval uppercase_RESULT:=$(call uppercase_internal,$(uppercase_TABLE),$1))$(uppercase_RESULT)

PACKAGEUC = $(call uppercase,$(PACKAGE))
PACKAGESORTED = $(sort $(PACKAGEUC))


lmpinstalledpkgs.h: $(SRC) $(INC)
	@echo '#ifndef LMP_INSTALLED_PKGS_H' >  ${TMPNAME}.lmpinstalled
	@echo '#define LMP_INSTALLED_PKGS_H' >> ${TMPNAME}.lmpinstalled
	@echo 'const char * LAMMPS_NS::LAMMPS::installed_packages[] = {' >> ${TMPNAME}.lmpinstalled
	@for p in $(PACKAGEUC); do info=$$($(SHELL) Package.sh $$p installed); \
             [ -n "$$info" ] && echo "\"$$info\"" | sed -e 's/".*package \(.*\)"/"\1",/' >> ${TMPNAME}.lmpinstalled || :; done
	@echo ' NULL };' >> ${TMPNAME}.lmpinstalled
	@echo '#endif' >> ${TMPNAME}.lmpinstalled
	@if [ -f lmpinstalledpkgs.h ]; \
          then test "`diff --brief ${TMPNAME}.lmpinstalled lmpinstalledpkgs.h`" != "" && \
	        mv ${TMPNAME}.lmpinstalled lmpinstalledpkgs.h || rm ${TMPNAME}.lmpinstalled ; \
        else mv ${TMPNAME}.lmpinstalled lmpinstalledpkgs.h ; fi

gitversion:
	@echo 'Gathering git version information'
	@echo '#ifndef LMP_GIT_VERSION_H' >  ${TMPNAME}.lmpgitversion
	@echo '#define LMP_GIT_VERSION_H' >> ${TMPNAME}.lmpgitversion
	@if (type git && test -e ../.git ) >> /dev/null 2>> /dev/null ; then \
	  git='true';					\
	  commit=$$(git rev-parse HEAD);		\
	  branch=$$(git rev-parse --abbrev-ref HEAD);	\
	  describe=$$(git describe --dirty=-modified);	\
	else \
	  git='false' ;					\
	  commit='(unknown)' ;				\
	  branch='(unknown)' ;				\
	  describe='(unknown)' ;			\
	fi ; \
	echo "bool LAMMPS_NS::LAMMPS::has_git_info() { return $${git}; }"        >> ${TMPNAME}.lmpgitversion ; \
	echo "const char *LAMMPS_NS::LAMMPS::git_commit() { return \"$${commit}\"; }" >> ${TMPNAME}.lmpgitversion ; \
	echo "const char *LAMMPS_NS::LAMMPS::git_branch() { return \"$${branch}\"; }" >> ${TMPNAME}.lmpgitversion ; \
	echo "const char *LAMMPS_NS::LAMMPS::git_descriptor() { return \"$${describe}\"; }" >> ${TMPNAME}.lmpgitversion
	@echo '#endif' >> ${TMPNAME}.lmpgitversion
	@if [ -f lmpgitversion.h ]; \
          then test "`diff --brief ${TMPNAME}.lmpgitversion lmpgitversion.h`" != "" && \
	        mv ${TMPNAME}.lmpgitversion lmpgitversion.h || rm ${TMPNAME}.lmpgitversion ; \
        else mv ${TMPNAME}.lmpgitversion lmpgitversion.h ; fi

# Build LAMMPS in one of 2 modes
# static = static compile in Obj_machine (default)
# shared = shared compile in Obj_shared_machine

.DEFAULT:
	@if [ $@ = "serial" ]; \
	  then cd STUBS; $(MAKE); cd ..; fi
	@test -f MAKE/Makefile.$@ -o -f MAKE/OPTIONS/Makefile.$@ -o \
	  -f MAKE/MACHINES/Makefile.$@ -o -f MAKE/MINE/Makefile.$@
	@if [ ! -d $(objdir) ]; then mkdir $(objdir); fi
	@echo 'Gathering installed package information (may take a little while)'
	@$(SHELL) Make.sh style
	@$(SHELL) Make.sh packages
	@$(MAKE) $(MFLAGS) lmpinstalledpkgs.h gitversion
	@echo 'Compiling LAMMPS for machine $@'
	@if [ -f MAKE/MACHINES/Makefile.$@ ]; \
	  then cp MAKE/MACHINES/Makefile.$@ $(objdir)/Makefile; fi
	@if [ -f MAKE/OPTIONS/Makefile.$@ ]; \
	  then cp MAKE/OPTIONS/Makefile.$@ $(objdir)/Makefile; fi
	@if [ -f MAKE/Makefile.$@ ]; \
	  then cp MAKE/Makefile.$@ $(objdir)/Makefile; fi
	@if [ -f MAKE/MINE/Makefile.$@ ]; \
	  then cp MAKE/MINE/Makefile.$@ $(objdir)/Makefile; fi
	@if [ ! -e Makefile.package ]; \
	  then cp Makefile.package.empty Makefile.package; fi
	@if [ ! -e Makefile.package.settings ]; \
	  then cp Makefile.package.settings.empty Makefile.package.settings; fi
	@cp Makefile.package Makefile.package.settings $(objdir)
	@cd $(objdir); rm -f .depend; \
	$(MAKE) $(MFLAGS) "SRC = $(SRC)" "INC = $(INC)" depend || :
	@rm -f $(ARLINK) $(SHLINK) $(EXE)
ifeq ($(mode),static)
	@cd $(objdir); \
	$(MAKE) $(MFLAGS) "OBJ = $(OBJLIB)" "INC = $(INC)" "SHFLAGS =" \
	  "LMPLIB = $(ARLIB)" "ARLIB = $(ARLIB)" "SHLIB = $(SHLIB)" \
	  "LMPLINK = $(LMPLINK)" "DYN_LIB = $(DYN_LIB)" "EXE = ../$(EXE)" ../$(EXE)
	@ln -s $(ARLIB) $(ARLINK)
endif
ifeq ($(mode),shared)
	@cd $(objdir); \
	$(MAKE) $(MFLAGS) "OBJ = $(OBJLIB)" "INC = $(INC)" \
	  "LMPLIB = $(SHLIB)" "ARLIB = $(ARLIB)" "SHLIB = $(SHLIB)" \
	  "LMPLINK = $(LMPLINK)" "DYN_LIB = $(DYN_LIB)" "EXE = ../$(EXE)" ../$(EXE)
	@ln -s $(SHLIB) $(SHLINK)
endif
# backward compatibility
ifeq ($(mode),exe)
	$(MAKE) $(MFLAGS) mode=static $@
endif
ifeq ($(mode),lib)
	$(MAKE) $(MFLAGS) mode=static $@
endif
ifeq ($(mode),shexe)
	$(MAKE) $(MFLAGS) mode=shared $@
endif
ifeq ($(mode),shlib)
	$(MAKE) $(MFLAGS) mode=shared $@
endif

ifeq ($(mode),print)
	@cd $(objdir); \
	$(MAKE) $(MFLAGS) "OBJ = $(OBJLIB)" "INC = $(INC)" \
	  "EXE = ../$(ARLIB)" -f ../Makefile.print
endif

clean-all:
	rm -rf Obj_*
	rm -f style_*.h packages_*.h lmpgitversion.h lmpinstalledpkgs.h
