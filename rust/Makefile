.PHONY: build run test clean lint fmt check bench

build:
	cargo build --release

run:
	cargo run --release

test:
	cargo test --verbose

test-all:
	cargo test --all-features --verbose

lint:
	cargo clippy -- -D warnings

fmt:
	cargo fmt --all

fmt-check:
	cargo fmt --all -- --check

check:
	cargo check

bench:
	cargo bench

clean:
	cargo clean

coverage:
	cargo tarpaulin --out Html --output-dir coverage
