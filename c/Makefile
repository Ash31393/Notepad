.PHONY: build run test clean format check

build:
	mkdir -p build
	cd build && cmake .. && cmake --build .

run: build
	./build/notepad

test: build
	cd build && ctest --verbose

format:
	clang-format -i src/*.c include/*.h

check:
	clang-tidy src/*.c -- -I include

clean:
	rm -rf build

coverage: build
	cd build && make coverage
