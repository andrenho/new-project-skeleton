VERSION = 0.0.1

VPATH := src
OBJS := main.o

#
# compilation options
#
CPPFLAGS = -std=c++14 -DVERSION=\"$(VERSION)\"

#
# add cflags/libraries
#
CPPFLAGS += #`pkg-config --cflags sdl2`
LDFLAGS  += #-fuse-ld=gold `pkg-config --libs sdl2` -lBox2D

#
# add warnings
#
ifndef W
  WARNINGS := @build/warnings.txt
  ifeq ($(CXX),g++)
    WARNINGS += @build/warnings_gcc.txt
  endif
endif

# 
# debug/release
# 
all:
	@echo Choose a target: 'debug' or 'release'.

debug: TARGET_CXXFLAGS = -g -ggdb3 -O0 -DDEBUG -fno-inline-functions
debug: TARGET_LDFLAGS = -g
debug: myexecutable

release: TARGET_CXXFLAGS = -DNDEBUG -Ofast -fomit-frame-pointer -ffast-math -mfpmath=sse -fPIC -msse -msse2 -msse3 -mssse3 -msse4 -flto
release: TARGET_LDFLAGS = -flto -Wl,--strip-all
release: myexecutable

profile: TARGET_CXXFLAGS = -g -ggdb3 -O0 -DDEBUG -fno-inline-functions -pg
profile: TARGET_LDFLAGS = -g -pg
profile: myexecutable

#
# pull dependence info from existing .o files
#
-include $(OBJS:.o=.d)

# 
# compile source files
#
%.o: %.cc
	$(CXX) -c $(CPPFLAGS) $(CXXFLAGS) $(WARNINGS) $(TARGET_CXXFLAGS) $<
	$(CXX) -MM $(CPPFLAGS) $(CXXFLAGS) $< > $*.d
	@mv -f $*.d $*.d.tmp
	@sed -e 's|.*:|$*.o:|' < $*.d.tmp > $*.d
	@cp -f $*.d $*.d.tmp
	@sed -e 's/.*://' -e 's/\\$$//' < $*.d.tmp | fmt -1 | \
	  sed -e 's/^ *//' -e 's/$$/:/' >> $*.d
	@rm -f $*.d.tmp

#
# link
#
myexecutable: $(OBJS)
	$(CXX) $^ -o $@ $(TARGET_LDFLAGS) $(LDFLAGS)

#
# other rules
#
cloc:
	cloc Makefile src/*.hh src/*.cc

lint:
	clang-tidy src/*.hh src/*.cc "-checks=*,-google-build-using-namespace,-google-readability-todo" -- -I. --std=c++14 -DVERSION=\"$(VERSION)\"

check-leaks: debug
	valgrind --leak-check=full --show-leak-kinds=all --track-origins=yes --suppressions=build/myexecutable.supp ./myexecutable

gen-suppressions: debug
	valgrind --leak-check=full --show-leak-kinds=all --track-origins=yes --error-limit=no --gen-suppressions=all --log-file=build/myexecutable.supp ./myexecutable
	sed -i -e '/^==.*$$/d' build/myexecutable.supp

clean:
	rm -f myexecutable *.o *.d

.PHONY: debug release profile cloc lint check-leaks gen-suppressions clean
