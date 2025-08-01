# Architecture Documentation

## System Overview

The CSV Bank Statement Merger is designed as a modular, extensible CLI application that processes financial data with high reliability and performance. The architecture follows clean separation of concerns with well-defined interfaces between components.

## Core Principles

- **Data Integrity First**: All transformations preserve original data accuracy
- **Extensible Design**: New bank formats can be added without core changes  
- **Performance Focus**: Efficient processing of large CSV files
- **Error Resilience**: Graceful handling of malformed or unexpected data
- **Security by Design**: Local processing with no network transmission

## High-Level Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   CLI Layer     │───▶│  Business Logic  │───▶│  Data Layer     │
│                 │    │                  │    │                 │
│ • Command Parse │    │ • Format Detection│    │ • CSV Parser    │
│ • Input Valid   │    │ • Data Transform  │    │ • File I/O      │
│ • Output Format │    │ • Merge Logic     │    │ • Validation    │
└─────────────────┘    └──────────────────┘    └─────────────────┘
```

## Module Architecture

### 1. CLI Module (`src/cli/`)

**Purpose**: Handle user interaction and command processing

**Components**:
- `commands.rs`: Command definitions and argument parsing
- `validation.rs`: Input validation and error reporting
- `output.rs`: Result formatting and file writing

**Key Responsibilities**:
- Parse command-line arguments using `clap`
- Validate input files and directories
- Coordinate processing workflow
- Format and display results/errors

### 2. Parser Module (`src/parser/`)

**Purpose**: Handle CSV file reading and initial data extraction

**Components**:
- `generic.rs`: Base CSV parsing functionality
- `detector.rs`: Automatic format detection logic
- `adapters/`: Bank-specific parsing implementations
  - `chase.rs`: Chase Bank CSV format
  - `wells_fargo.rs`: Wells Fargo format
  - `bank_of_america.rs`: BofA format
  - `generic_configurable.rs`: User-defined formats

**Key Responsibilities**:
- Read CSV files efficiently
- Detect bank format automatically or via configuration
- Transform raw CSV data into internal representation
- Handle encoding and delimiter variations

### 3. Models Module (`src/models/`)

**Purpose**: Define core data structures and business entities

**Components**:
- `transaction.rs`: Individual transaction representation
- `statement.rs`: Bank statement container
- `config.rs`: Configuration structures
- `errors.rs`: Custom error types

**Core Data Model**:
```rust
pub struct Transaction {
    pub date: NaiveDate,
    pub description: String,
    pub amount: Decimal,
    pub balance: Option<Decimal>,
    pub category: Option<String>,
    pub account_id: String,
    pub transaction_id: Option<String>,
    pub metadata: HashMap<String, String>,
}

pub struct Statement {
    pub bank_name: String,
    pub account_number: String,
    pub statement_period: DateRange,
    pub transactions: Vec<Transaction>,
    pub opening_balance: Option<Decimal>,
    pub closing_balance: Option<Decimal>,
}
```

### 4. Merger Module (`src/merger/`)

**Purpose**: Consolidate data from multiple sources

**Components**:
- `consolidator.rs`: Main merging logic
- `deduplication.rs`: Duplicate transaction detection
- `reconciliation.rs`: Balance and consistency checks
- `export.rs`: Output format generation

**Key Responsibilities**:
- Merge transactions from multiple statements
- Detect and handle duplicate transactions
- Maintain chronological ordering
- Validate merged data integrity

## Data Flow Architecture

### Processing Pipeline

```
Input Files → Format Detection → Parsing → Normalization → Merging → Validation → Output
     │              │              │           │            │           │          │
     │              │              │           │            │           │          │
     ▼              ▼              ▼           ▼            ▼           ▼          ▼
CSV Files    Bank Adapters   Raw Records  Transactions  Merged Set   Checks   Export
```

### Detailed Flow

1. **Input Stage**
   - File discovery and validation
   - Format detection (automatic or manual)
   - Configuration loading

2. **Parsing Stage**
   - CSV parsing with appropriate adapter
   - Data type conversion and validation
   - Error collection and reporting

3. **Transformation Stage**
   - Normalize to common Transaction model
   - Apply field mappings and transformations
   - Validate business rules

4. **Merging Stage**
   - Combine transactions from all sources
   - Sort chronologically
   - Deduplicate based on configurable rules
   - Reconcile balances where possible

5. **Output Stage**
   - Generate requested output format
   - Apply filters or transformations
   - Write to specified destination

## Error Handling Strategy

### Error Categories

1. **User Errors**: Invalid arguments, missing files
2. **Data Errors**: Malformed CSV, invalid dates/amounts  
3. **System Errors**: File permissions, memory issues
4. **Business Logic Errors**: Inconsistent balances, duplicate detection failures

### Error Recovery

- **Graceful Degradation**: Continue processing valid data when possible
- **Detailed Reporting**: Comprehensive error messages with context
- **Partial Success**: Allow output of successfully processed data
- **Rollback Capability**: Maintain original data integrity

## Configuration Architecture

### Configuration Hierarchy

1. **Built-in Defaults**: Hard-coded sensible defaults
2. **Global Config**: System-wide configuration file
3. **Project Config**: Per-project configuration file  
4. **Command Line**: Runtime parameter overrides

### Configuration Structure

```toml
[general]
default_output_format = "csv"
date_format = "%Y-%m-%d"
decimal_precision = 2

[formats.chase]
date_column = "Transaction Date"
description_column = "Description"
amount_column = "Amount"
balance_column = "Balance"

[deduplication]
enabled = true
match_fields = ["date", "amount", "description"]
tolerance_days = 1
amount_tolerance = 0.01
```

## Security Architecture

### Data Protection

- **Local Processing Only**: No network communication
- **Memory Management**: Secure cleanup of sensitive data
- **File Permissions**: Respect system access controls
- **Audit Trail**: Optional logging of processing steps

### Input Validation

- **Path Traversal Prevention**: Validate file paths
- **CSV Injection Protection**: Sanitize formula-like content
- **Size Limits**: Prevent resource exhaustion attacks
- **Encoding Validation**: Handle only safe character encodings

## Performance Architecture

### Optimization Strategies

1. **Streaming Processing**: Process large files without loading entirely into memory
2. **Parallel Processing**: Concurrent parsing of multiple files
3. **Lazy Evaluation**: Load and process data on-demand
4. **Memory Pool**: Reuse allocated memory for better performance

### Scalability Considerations

- **Memory Usage**: Linear scaling with file size, not quadratic
- **CPU Utilization**: Multi-threaded processing where beneficial
- **I/O Optimization**: Efficient file reading with appropriate buffer sizes
- **Caching**: Cache parsed configuration and format definitions

## Extension Points

### Adding New Bank Formats

1. Implement `BankAdapter` trait
2. Add format detection rules
3. Create comprehensive tests
4. Update documentation

### Custom Output Formats

1. Implement `OutputFormatter` trait
2. Add CLI option for new format
3. Update help documentation

### Plugin Architecture (Future)

- Dynamic loading of format adapters
- External validation rule definitions
- Custom transformation functions

## Dependencies Architecture

### Core Dependencies

- **csv**: Fast, standards-compliant CSV parsing
- **serde**: Serialization/deserialization framework
- **clap**: Command-line argument parsing
- **chrono**: Date/time handling
- **rust_decimal**: Precise decimal arithmetic for financial data

### Development Dependencies

- **tokio-test**: Async testing utilities
- **proptest**: Property-based testing
- **criterion**: Benchmarking framework
- **tarpaulin**: Code coverage analysis

## Deployment Architecture

### Build Targets

- **Native Binary**: Platform-specific optimized executable
- **Cross-Platform**: Support for Windows, macOS, Linux
- **Container**: Docker image for isolated execution
- **Library**: Reusable crate for integration

### Distribution

- **Cargo**: Published to crates.io
- **GitHub Releases**: Pre-built binaries
- **Package Managers**: Homebrew, APT, etc.
- **Enterprise**: Internal distribution mechanisms