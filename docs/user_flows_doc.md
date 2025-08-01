# User Flows

## Flow Overview

This document describes the primary user journeys through the CSV Bank Statement Merger application, from initial setup through various usage scenarios and error handling paths.

## Primary User Personas

### 1. Sarah - Personal Finance Enthusiast
- **Background**: Individual with accounts at multiple banks
- **Goal**: Monthly financial analysis and budgeting
- **Technical Level**: Basic command-line comfort
- **Frequency**: Monthly usage

### 2. Mike - Small Business Owner  
- **Background**: Manages business accounts across 2-3 banks
- **Goal**: Quarterly financial reconciliation
- **Technical Level**: Moderate technical skills
- **Frequency**: Quarterly usage with batch processing

### 3. Lisa - Financial Analyst
- **Background**: Processes client financial data
- **Goal**: Client reporting and analysis
- **Technical Level**: Advanced technical user
- **Frequency**: Daily usage with various formats

## Core User Flows

### Flow 1: First-Time Setup and Basic Merge

**User**: Sarah (Personal Finance Enthusiast)

#### Pre-conditions
- Has downloaded CSV statements from Chase and Wells Fargo
- Has installed the csv-bank-merger tool
- Files are in ~/Downloads/ folder

#### Flow Steps

1. **Discovery Phase**
   ```bash
   # Sarah explores the tool
   csv-bank-merger --help
   ```
   - **System**: Displays comprehensive help with examples
   - **User Action**: Reviews available options
   - **Expected Outcome**: Understands basic usage pattern

2. **Initial Command Attempt**
   ```bash
   # Sarah tries basic merge
   csv-bank-merger chase_jan.csv wells_fargo_jan.csv
   ```
   - **System**: Shows error - no output file specified
   - **User Action**: Reads error message
   - **Expected Outcome**: Understands --output is required

3. **Successful Basic Merge**
   ```bash
   # Sarah specifies output file
   csv-bank-merger --output january_merged.csv chase_jan.csv wells_fargo_jan.csv
   ```
   - **System**: 
     - Detects Chase format in first file
     - Detects Wells Fargo format in second file
     - Processes 234 transactions from Chase
     - Processes 156 transactions from Wells Fargo
     - Shows progress: "Processing chase_jan.csv... ✓"
     - Shows progress: "Processing wells_fargo_jan.csv... ✓"
     - Shows summary: "Merged 390 transactions from 2 sources"
   - **User Action**: Reviews console output
   - **Expected Outcome**: january_merged.csv created successfully

4. **Verification Phase**
   ```bash
   # Sarah examines the output
   head january_merged.csv
   ```
   - **System**: N/A (external command)
   - **User Action**: Verifies merged data looks correct
   - **Expected Outcome**: Sees unified format with Date, Description, Amount, Account, Bank columns

#### Success Criteria
- ✅ Merged file created without errors
- ✅ All transactions preserved with source identification
- ✅ Data format is suitable for Excel import
- ✅ Process completed in under 10 seconds

#### Alternative Paths

**Path 1A: Automatic Directory Processing**
```bash
# Sarah discovers directory option
csv-bank-merger --input ~/Downloads/bank_statements/ --output merged.csv
```
- Processes all CSV files in directory
- Shows file-by-file progress
- Creates comprehensive merged output

**Path 1B: Format Override**
```bash
# If auto-detection fails
csv-bank-merger --format chase --output merged.csv mystery_file.csv
```
- Uses specified format instead of auto-detection
- Warns if format doesn't match data structure

---

### Flow 2: Batch Processing for Business Use

**User**: Mike (Small Business Owner)

#### Pre-conditions
- Has Q1 statements for business checking, savings, and credit card
- Files follow naming convention: bank_account_month.csv
- Needs consolidated report for accountant

#### Flow Steps

1. **Organization Phase**
   ```bash
   # Mike organizes files
   ls q1_statements/
   # chase_checking_jan.csv, chase_checking_feb.csv, chase_checking_mar.csv
   # bofa_savings_jan.csv, bofa_savings_feb.csv, bofa_savings_mar.csv
   # chase_credit_jan.csv, chase_credit_feb.csv, chase_credit_mar.csv
   ```

2. **Batch Processing**
   ```bash
   # Process entire quarter
   csv-bank-merger --input q1_statements/ --output q1_consolidated.csv --verbose
   ```
   - **System**: 
     - Scans directory: "Found 9 CSV files"
     - Processes each file with detailed progress
     - Shows format detection for each: "chase_checking_jan.csv → Chase format"
     - Reports transaction counts per file
     - Shows final summary: "Processed 2,847 transactions from 9 files"
   - **User Action**: Monitors verbose output for any issues
   - **Expected Outcome**: Single consolidated file with all Q1 transactions

3. **Quality Check**
   ```bash
   # Mike reviews processing results
   csv-bank-merger --input q1_statements/ --output /dev/null --dry-run
   ```
   - **System**: 
     - Validates all files without creating output
     - Reports any data quality issues
     - Shows what would be processed
   - **User Action**: Reviews validation results
   - **Expected Outcome**: Confidence in data quality before sending to accountant

#### Success Criteria
- ✅ All 9 files processed successfully
- ✅ Transactions properly sorted chronologically
- ✅ Account information preserved for tracking
- ✅ No data loss or corruption

#### Alternative Paths

**Path 2A: Error Recovery**
```bash
# If one file has issues
csv-bank-merger --input q1_statements/ --output q1_partial.csv --continue-on-error
```
- Processes valid files, skips problematic ones
- Reports which files were skipped and why
- Allows partial success rather than complete failure

---

### Flow 3: Advanced Processing with Custom Configuration

**User**: Lisa (Financial Analyst)

#### Pre-conditions
- Processes statements from various regional banks
- Has custom CSV format that doesn't auto-detect
- Needs specific output formatting for reporting system

#### Flow Steps

1. **Format Analysis**
   ```bash
   # Lisa examines unknown format
   head unknown_bank.csv
   # Date,Desc,Debit,Credit,Running Balance
   # 01/15/2024,"DEPOSIT ATM",,"1500.00","2750.00"
   ```

2. **Generic Format Processing**
   ```bash
   # Process with manual field mapping
   csv-bank-merger --format generic \
     --date-column "Date" \
     --description-column "Desc" \
     --amount-columns "Debit,Credit" \
     --output processed.csv \
     unknown_bank.csv
   ```
   - **System**: 
     - Uses generic parser with specified mappings
     - Combines debit/credit columns into signed amounts
     - Processes 445 transactions
     - Reports successful conversion
   - **User Action**: Verifies output format matches expectations
   - **Expected Outcome**: Successfully processed custom format

3. **Integration with Workflow**
   ```bash
   # Combine with other standard formats
   csv-bank-merger --output client_report.csv \
     processed.csv \
     chase_client.csv \
     wells_fargo_client.csv
   ```
   - **System**: Merges pre-processed file with auto-detected formats
   - **User Action**: Generates final client report
   - **Expected Outcome**: Unified report ready for analysis

#### Success Criteria
- ✅ Custom format successfully parsed
- ✅ Data integrity maintained through conversion
- ✅ Output compatible with downstream systems
- ✅ Repeatable process for similar files

---

## Error Handling Flows

### Error Flow 1: Invalid File Format

#### Scenario
User attempts to process a non-CSV file or corrupted CSV

```bash
csv-bank-merger --output result.csv invalid_file.txt malformed.csv
```

#### System Response
1. **File Validation**
   - Detects invalid_file.txt is not CSV format
   - Shows clear error: "Error: invalid_file.txt does not appear to be a valid CSV file"
   - Continues processing malformed.csv

2. **Parsing Errors**
   - Encounters corrupted rows in malformed.csv
   - Shows: "Warning: Skipped 3 invalid rows in malformed.csv"
   - Lists row numbers and reasons for skipping

3. **Completion with Warnings**
   - Processes valid data from malformed.csv
   - Shows final status: "Completed with warnings - see above for details"
   - Creates output file with successfully processed transactions
   - Returns exit code 1 to indicate warnings

#### User Response Options
- Review warnings and decide if results are acceptable
- Fix source files and re-run
- Use --strict mode to fail on any invalid data

### Error Flow 2: Format Detection Failure

#### Scenario
CSV file doesn't match any known bank format

```bash
csv-bank-merger --output result.csv mystery_bank.csv
```

#### System Response
1. **Detection Attempt**
   - Tries each known format adapter
   - Shows: "Warning: Could not auto-detect format for mystery_bank.csv"
   - Suggests: "Try specifying format manually with --format option"

2. **Fallback Options**
   - Offers to use generic parser
   - Shows available format options
   - Provides example of manual specification

#### User Response Flow
```bash
# User tries manual format specification
csv-bank-merger --format generic --output result.csv mystery_bank.csv

# Or examines file structure first
csv-bank-merger --analyze mystery_bank.csv
```

### Error Flow 3: Data Integrity Issues

#### Scenario
Inconsistent or suspicious data detected during processing

```bash
csv-bank-merger --output result.csv statements_with_issues.csv
```

#### System Response
1. **Data Validation**
   - Detects future dates: "Warning: Found 2 transactions with future dates"
   - Finds unusual amounts: "Warning: Transaction amount $999,999.99 seems unusually large"
   - Identifies missing data: "Warning: 5 transactions missing descriptions"

2. **User Consultation**
   - Asks: "Continue processing despite warnings? [y/N]"
   - Provides option to review issues first
   - Allows selective filtering of problematic data

#### User Decision Points
- Continue with warnings (data included as-is)
- Review and fix source data
- Apply automatic filtering for common issues

## Success Paths Summary

### Quick Success (90% of users)
1. Run basic command with files
2. Get merged output immediately
3. Use result in Excel or other tools

### Guided Success (8% of users)
1. Encounter minor format issues
2. Use help messages to adjust command
3. Successfully process with correct options

### Advanced Success (2% of users)
1. Use advanced options for custom formats
2. Integrate into automated workflows
3. Handle complex multi-source scenarios

## User Experience Principles

### Discoverability
- Help is always available with --help
- Error messages include suggested solutions
- Common usage patterns shown in examples

### Forgiveness
- Warnings don't stop processing
- Partial success is clearly communicated
- Easy recovery from common mistakes

### Transparency
- Processing steps are visible
- Data transformations are explained
- Quality issues are clearly reported

### Efficiency
- Default behavior handles common cases
- Advanced options available when needed
- Minimal typing required for standard workflows