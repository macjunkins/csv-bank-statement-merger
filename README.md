# CSV Bank Statement Merger

A robust Rust application that parses and merges bank statements from multiple CSV formats into a unified, standardized format.

## 🎯 Goals

- **Unify disparate formats**: Parse CSV bank statements from different financial institutions with varying column structures and data formats
- **Data standardization**: Convert all statements into a common, consistent format for easy analysis
- **Preserve data integrity**: Ensure accuracy and completeness during the parsing and merging process
- **Command-line efficiency**: Provide a fast, reliable CLI tool for batch processing multiple files
- **Extensible design**: Support for adding new bank formats without major code changes

## ✨ Features

- 📊 **Multi-format parsing**: Support for various CSV bank statement formats
- 🔄 **Smart merging**: Intelligent consolidation of data from multiple sources
- 🛡️ **Data validation**: Built-in checks to ensure data integrity and accuracy
- ⚡ **Performance**: Fast processing of large CSV files using Rust's efficiency
- 🎨 **Flexible output**: Export merged data in multiple formats (CSV, JSON)
- 🔧 **Configurable**: Customizable field mappings and transformation rules

## 🚀 Quick Start

### Prerequisites

- Rust (1.70.0 or later)
- Cargo package manager

### Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/csv-bank-statement-merger.git
cd csv-bank-statement-merger

# Build the project
cargo build --release

# Install globally (optional)
cargo install --path .
```

### Basic Usage

```bash
# Merge multiple bank statement files
csv-bank-merger --input statements/ --output merged_statements.csv

# Process specific files with custom format
csv-bank-merger --files bank1.csv bank2.csv --format json --output result.json

# Use configuration file for custom field mappings
csv-bank-merger --config config.toml --input statements/ --output merged.csv
```

## 📁 Project Structure

```
├── src/
│   ├── main.rs              # CLI entry point
│   ├── lib.rs               # Library root
│   ├── parser/              # CSV parsing modules
│   │   ├── mod.rs
│   │   ├── generic.rs       # Generic CSV parser
│   │   └── adapters/        # Bank-specific adapters
│   ├── merger/              # Data merging logic
│   │   ├── mod.rs
│   │   └── consolidator.rs
│   ├── models/              # Data structures
│   │   ├── mod.rs
│   │   ├── transaction.rs
│   │   └── statement.rs
│   └── cli/                 # Command-line interface
│       ├── mod.rs
│       └── commands.rs
├── tests/                   # Test files
├── examples/                # Example usage and sample data
├── docs/                    # Documentation
└── config/                  # Configuration templates
```

## 📊 Supported Formats

The application currently supports or plans to support:

- **Chase Bank**: Standard CSV export format
- **Wells Fargo**: Transaction history CSV
- **Bank of America**: Statement CSV format
- **Capital One**: Credit card and bank statements
- **Generic CSV**: Configurable format with custom field mapping
- *More formats coming soon...*

## 🛠️ Development

### Building from Source

```bash
# Debug build
cargo build

# Release build
cargo build --release

# Run tests
cargo test

# Run with example data
cargo run -- --input examples/sample_statements/ --output test_output.csv
```

### Adding New Bank Formats

1. Create a new adapter in `src/parser/adapters/`
2. Implement the `BankAdapter` trait
3. Add format detection logic
4. Include unit tests for the new format
5. Update documentation

## 🧪 Testing

```bash
# Run all tests
cargo test

# Run integration tests
cargo test --test integration

# Run with coverage (requires cargo-tarpaulin)
cargo tarpaulin --out Html
```

## 📚 Documentation

- [API Documentation](docs/api.md)
- [Configuration Guide](docs/configuration.md)
- [Adding New Formats](docs/extending.md)
- [Example Usage](examples/README.md)

## 🤝 Contributing

We welcome contributions! Please see our [Contributing Guide](CONTRIBUTING.md) for details.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🏗️ Roadmap

- [ ] **Phase 1**: Core CSV parsing and basic merging
- [ ] **Phase 2**: Support for 5+ major bank formats
- [ ] **Phase 3**: Web interface for non-technical users
- [ ] **Phase 4**: API service for integration with other tools
- [ ] **Phase 5**: Machine learning for automatic format detection

## 📊 Status

![Build Status](https://img.shields.io/github/workflow/status/yourusername/csv-bank-statement-merger/CI)
![License](https://img.shields.io/github/license/yourusername/csv-bank-statement-merger)
![Version](https://img.shields.io/crates/v/csv-bank-statement-merger)

## 🙏 Acknowledgments

- Built with [Rust](https://www.rust-lang.org/)
- CSV parsing powered by [csv](https://crates.io/crates/csv)
- CLI built with [clap](https://crates.io/crates/clap)
- Serialization with [serde](https://crates.io/crates/serde)

---

**⚠️ Security Notice**: This tool processes financial data locally and does not transmit any information over the network. Always review your data and use appropriate security measures when handling sensitive financial information.
