.PHONY: build test clean lint fmt vet

build:
	go build -o bin/notepad ./cmd/notepad

run:
	go run ./cmd/notepad

test:
	go test -v -cover ./...

test-race:
	go test -v -race ./...

lint:
	golangci-lint run ./...

fmt:
	gofmt -w .
	goimports -w .

vet:
	go vet ./...

clean:
	rm -rf bin/

coverage:
	go test -coverprofile=coverage.out ./...
	go tool cover -html=coverage.out
