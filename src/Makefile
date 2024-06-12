CC = gcc 
SANITIZE=-fsanitize=address
TST_LIBS += $(shell pkg-config --libs --cflags check) -lncurses

ifeq ($(shell uname), Darwin)
FLAGS =-Wall -Wextra -Werror -std=c11 -I /usr/local/include
else
ifeq ($(shell uname), Linux)
	FLAGS =-Wall -Wextra -Werror -std=gnu11
endif
endif

LIB_NAME = s21_tetris
BACK=$(wildcard brick_game/tetris/*.c)
OBJ_BACK=$(BACK:.c=.o)
TEST=$(wildcard brick_game/tetris/tests/tests.c)

all: install

# ===== CLASSIC BUILD =====
%.o: %.c
	@$(CC) $(FLAGS) -c $< -o $@

mkbuild:
	@rm -rf build/
	@mkdir build
	@touch build/record.txt

uninstall :
	@rm -rf build/
	@echo "\033[31mGame uninstalled \033[0m"

install : mkbuild main
	@echo "\033[32mGame installed \033[0m"

back_lib : $(OBJ_BACK) 
	@ar rc build/back.a $^
	@ranlib build/back.a 
	@rm brick_game/tetris/*.o

front_lib:
	@$(CC) $(FLAGS) -c gui/cli/frontend.c -o gui/cli/frontend.o
	@ar rc build/front.a gui/cli/frontend.o
	@ranlib build/front.a 
	@rm gui/cli/*.o

main: back_lib front_lib
	@$(CC) tetris.c build/*.a -o build/TETRIS -lncurses

# ===== DOCS =====
dvi:
	@doxygen .Doxyfile
	@open doxygen/html/index.html

# ===== ARCHIVE =====
dist: install
	@rm -rf dist.tar.gz
	@tar -cvzf dist.tar.gz build/Tetris

# ===== TESTS ===== 
add_coverage_flag:
	$(eval FLAGS += --coverage)

test : clean mkbuild back_lib front_lib
	$(CC) $(FLAGS) $(TEST) -lncurses build/back.a $(TST_LIBS) -o test

gcov_report: add_coverage_flag test
	./test
	
	lcov -t "check" -o check.info -c -d ./  --no-external $(LCOVFLAGS)
	genhtml -o report check.info $(LCOVFLAGS)
	open report/index.html

# ===== CHECKS ======
leaks: test
	leaks -atExit -- ./test | grep leaks

valgrind: install
	valgrind --tool=memcheck --leak-check=yes  ./build/TETRIS

cppcheck:
	cppcheck --enable=all --suppress=missingIncludeSystem .

style:
	cp ../materials/linters/.clang-format .
	clang-format -i brick_game/tetris/tests/*.c brick_game/tetris/*.c brick_game/tetris/*.h gui/cli/*.c gui/cli/*.h tetris.c brick_game/tetris/tests/*.c
	clang-format -n brick_game/tetris/tests/*.c brick_game/tetris/*.c brick_game/tetris/*.h gui/cli/*.c gui/cli/*.h tetris.c brick_game/tetris/tests/*.c
	rm -f .clang-format

sanitize : mkbuild back_lib front_lib
	@$(CC) $(FLAGS) $(SANITIZE) -lncurses build/back.a build/front.a tetris.c -o build/TETRIS 
	@echo "\033[32mSanitize done \033[0m"


# ===== CLEANING =====
clean :
	rm -rf  *.html TETRIS brick_game/tetris/*.o gui/cli/*.o build/ report/ *.dSYM doxygen/ brick_game/tetris/*.gcda gui/cli/*.gcda gui/cli/*.gcno brick_game/tetris/*.gcno *.gcda *gcno *.info *.css .saveScore.txt test dist.tar.gz
