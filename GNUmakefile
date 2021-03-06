# Project variables
PROJECT_NAME := terraform-provider-cloudsigma
PKG_NAME = cloudsigma
WEBSITE_REPO = github.com/hashicorp/terraform-website

# Build variables
.DEFAULT_GOAL = test
BUILD_DIR := build
DEV_GOARCH := $(shell go env GOARCH)
DEV_GOOS := $(shell go env GOOS)


## tools: Install required tooling.
.PHONY: tools
tools:
	@echo "Installing required tooling..."
	@curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- -b .bin/ v1.24.0


## clean: Delete the build directory.
.PHONY: clean
clean:
	@echo "==> Removing '$(BUILD_DIR)' directory..."
	@rm -rf $(BUILD_DIR)


## lint: Lint code with golangci-lint.
.PHONY: lint
lint:
	@echo "==> Linting code with 'golangci-lint'..."
	@.bin/golangci-lint run ./...


## fmtcheck: internal task required by tf-deploy compile
.PHONY: fmtcheck
fmtcheck:
	@sh -c "'$(CURDIR)/scripts/gofmtcheck.sh'"


## test: Run all unit tests.
.PHONY: test
test: fmtcheck
	@echo "==> Running unit tests..."
	@mkdir -p $(BUILD_DIR)
	@go test -v -cover -coverprofile=$(BUILD_DIR)/coverage.out -parallel=4 ./...


## testacc: Run all acceptance tests.
.PHONY: testacc
testacc: fmtcheck
	@echo "==> Running acceptance tests..."
	TF_ACC=1 go test $(TEST) -v $(TESTARGS) -timeout 120m ./...


## build: Build binary for default local system's operating system and architecture.
.PHONY: build
build:
	@echo "==> Building binary..."
	@echo "    running go build for GOOS=$(DEV_GOOS) GOARCH=$(DEV_GOARCH)"
# workaround for missing .exe extension on Windows
ifeq ($(OS),Windows_NT)
	@go build -o $(BUILD_DIR)/$(PROJECT_NAME).exe main.go
else
	@go build -o $(BUILD_DIR)/$(PROJECT_NAME) main.go
endif


## website: Build website for the provider.
website:
.PHONY: website
ifeq (,$(wildcard $(GOPATH)/src/$(WEBSITE_REPO)))
	@echo "$(WEBSITE_REPO) not found in your GOPATH (necessary for layouts and assets), get-ting..."
	@git clone https://$(WEBSITE_REPO) $(GOPATH)/src/$(WEBSITE_REPO)
endif
	@echo "==> Building website for CloudSigma provider..."
	@$(MAKE) -C $(GOPATH)/src/$(WEBSITE_REPO) website-provider PROVIDER_PATH=$(shell pwd) PROVIDER_NAME=$(PKG_NAME)

## website-test: Check website for the provider.
.PHONY: website-test
website-test:
ifeq (,$(wildcard $(GOPATH)/src/$(WEBSITE_REPO)))
	@echo "$(WEBSITE_REPO) not found in your GOPATH (necessary for layouts and assets), get-ting..."
	@git clone https://$(WEBSITE_REPO) $(GOPATH)/src/$(WEBSITE_REPO)
endif
	@echo "==> Checking website for CloudSigma provider..."
	@$(MAKE) -C $(GOPATH)/src/$(WEBSITE_REPO) website-provider-test PROVIDER_PATH=$(shell pwd) PROVIDER_NAME=$(PKG_NAME)


help: GNUmakefile
	@echo "Usage: make <command>"
	@echo ""
	@echo "Commands:"
	@sed -n 's/^##//p' $< | column -t -s ':' |  sed -e 's/^/ /'
