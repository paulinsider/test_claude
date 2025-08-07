# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Nacos (Dynamic **Na**ming and **Co**nfiguration **S**ervice) is an easy-to-use platform designed for dynamic service discovery, configuration management, and service management for cloud-native applications and microservices.

## Build & Development Commands

### Prerequisites
- Java 17+ (required for compilation and runtime)
- Maven 3.6.3+ (distribution management and packaging)

### Build Commands
```bash
# Build entire project with release profile
mvn -Prelease-nacos -Dmaven.test.skip=true clean install -U

# Alternative using Maven wrapper
./mvnw -Prelease-nacos -Dmaven.test.skip=true clean install -U

# Run tests
mvn test

# Build specific integration test profiles
mvn test -Pcit-test    # Configuration integration tests
mvn test -Pnit-test    # Naming integration tests
```

### Code Quality & Analysis
```bash
# PMD static analysis (automatically runs on build)
mvn pmd:check

# Checkstyle validation (runs during validate phase)
mvn validate

# Generate code coverage reports
mvn test jacoco:report

# SpotBugs analysis
mvn spotbugs:check
```

### Console UI Development
```bash
cd console-ui/
npm start          # Development server
npm run build      # Production build
npm run eslint     # Lint JavaScript files
npm run eslint-fix # Auto-fix linting issues
```

### Server Startup (Development)
```bash
# From distribution/bin directory
sh startup.sh -m standalone    # Linux/Mac standalone mode
startup.cmd -m standalone      # Windows standalone mode
```

## Module Architecture

Nacos is organized as a multi-module Maven project with clear separation of concerns:

### Core Modules
- **api**: Public APIs and contracts for all services (config, naming, remote communication)
- **core**: Central server framework including cluster management, remote communication, auth, and distributed consensus
- **config**: Configuration management service with version control, gray deployment, and change notifications
- **naming**: Service discovery and registration with health checks and load balancing
- **common**: Shared utilities, constants, and base classes

### Infrastructure Modules
- **persistence**: Data access layer with pluggable database support (Derby, MySQL)
- **consistency**: Distributed consensus protocols (JRaft implementation)
- **auth**: Authentication and authorization framework with pluggable implementations
- **sys**: System configuration and environment management

### Client & Integration
- **client**: Java SDK for service consumers
- **client-basic**: Core client functionality shared across implementations
- **example**: Sample applications and usage examples
- **console**: Web-based management interface
- **console-ui**: React-based frontend application

### Plugin System
- **plugin**: Plugin framework definitions and interfaces
- **plugin-default-impl**: Default implementations for auth, control, and other plugins
- **lock**: Distributed locking service
- **ai**: AI and MCP (Model Context Protocol) integration features

### Distribution & Extensions
- **distribution**: Packaging and deployment configurations
- **bootstrap**: Application bootstrap and startup logic
- **server**: Server assembly and web configuration
- **test**: Integration test suites organized by module
- **istio**: Service mesh integration for Kubernetes environments
- **k8s-sync**: Kubernetes service synchronization
- **prometheus**: Metrics collection and monitoring integration

## Key Architecture Patterns

### Service Registry Pattern
- Services register instances with metadata
- Client-side load balancing with health checks
- Push-based change notifications to subscribers

### Configuration Center Pattern
- Hierarchical configuration with namespace isolation
- Real-time configuration push with change listeners
- Version control with rollback capabilities
- Gray deployment support for safe configuration updates

### Distributed Consensus
- JRaft-based consensus for configuration data consistency
- Distro protocol for AP-mode service discovery
- Pluggable consistency protocols

### Plugin Architecture
- SPI-based plugin system for extensibility
- Authentication, control, and data source plugins
- Hot-swappable implementations

### Multi-Protocol Communication
- HTTP/HTTPS REST APIs for web clients
- gRPC for high-performance client-server communication
- Long polling for efficient configuration updates

## Development Guidelines

### Module Dependencies
- Follow dependency hierarchy: client → api → common
- Core modules can depend on persistence, consistency, and sys
- Plugins should only depend on their respective plugin interfaces
- Avoid circular dependencies (enforced by Maven plugin)

### Configuration Management
- Use `nacos-*.properties` files for module-specific configuration
- Environment-specific properties in `application.properties`
- Plugin configurations in `META-INF/services/`

### Testing Strategy
- Unit tests in each module's `src/test/java`
- Integration tests in dedicated `test/` modules by functionality
- Use `*ITCase.java` naming for integration tests
- Separate profiles for different test categories

### Code Quality
- PMD rules based on Alibaba Java coding guidelines
- Checkstyle validation with custom rules
- Mandatory code coverage for new features
- SpotBugs for additional static analysis

## Common Development Patterns

When adding new features, follow established patterns:

1. **New API endpoints**: Add to appropriate controller in `*-controller` packages
2. **Configuration properties**: Use `@Value` or `@ConfigurationProperties` with appropriate defaults  
3. **Plugin implementations**: Implement SPI interfaces and register in `META-INF/services/`
4. **Database operations**: Use repository pattern with separate embedded/external implementations
5. **Event handling**: Use Spring's application event publisher for loose coupling
6. **Remote communication**: Extend existing request/response classes in `remote` packages