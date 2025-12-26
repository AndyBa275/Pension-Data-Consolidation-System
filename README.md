# Pension Data Consolidation and Quality Resolution System

## Executive Summary

An enterprise-grade data quality system that processed 3,765,847 pension contribution records from 1,133 heterogeneous Excel files, implementing advanced deduplication algorithms and graph-based consolidation to ensure data integrity for pension benefit calculations. The system successfully resolved systematic identifier conflicts and validated data against official government databases.

## Problem Statement

The pension scheme database contained critical data quality issues that prevented accurate benefit calculations:

### Part 1: Employee Number Conflicts
- Multiple individuals sharing the same employee identification numbers
- Single individuals possessing multiple employee numbers
- Missing and invalid employee identifiers (30 distinct cases)
- Systematic misuse of employee numbers across 3.7 million contribution records

### Part 2: Social Security Number (SSNIT) Quality Issues
- Missing SSNIT numbers preventing member identification
- Duplicate SSNIT numbers assigned to different individuals
- Format inconsistencies (temporary vs permanent, trailing zeros)
- Unverified records requiring validation against official government database

## Solution Architecture

### Technical Approach

The solution implements a two-phase data quality resolution system:

**Phase 1: Employee Number Deduplication**
- Multi-stage algorithmic approach to resolve identifier conflicts
- Fuzzy name matching with 80% similarity threshold
- SSNIT validation using pattern recognition
- Winner-selection algorithm based on record frequency
- Automated propagation of corrections across 2,905 monthly schedules

**Phase 2: SSNIT Quality Resolution**
- Graph-based Union-Find algorithm for transitive connection detection
- Fuzzy name matching with 85% similarity threshold for official database validation
- Format normalization addressing trailing zero anomalies
- Verification against 54,706 official government records

## Key Results

### Part 1: Employee Number Resolution
- **Source Data**: 1,133 Excel files with 3,765,847 contribution records
- **Standardized Schedules**: 2,905 monthly schedules created
- **New Employee Numbers Generated**: 9,525 unique identifiers for shared cases
- **Final Consolidation**: 66,866 unique members identified
- **Data Accuracy**: 100% of schedules updated with corrected identifiers

### Part 2: SSNIT Quality Resolution
- **Initial Dataset**: 66,866 members from Phase 1
- **Final Unique Members**: 66,719 validated member profiles
- **Ready for Processing**: 44,932 members (67.4%)
- **Missing SSNIT Identified**: 1,253 members (1.9%)
- **Incorrect SSNIT Detected**: 111 members (0.2%) using SSNIT numbers belonging to others
- **Consolidation via Trailing Zeros**: 3,119 duplicate records merged

## Technologies and Algorithms

### Core Technologies
- **Python 3.x**: Primary processing engine
- **Pandas**: Large-scale data manipulation and ETL operations
- **FuzzyWuzzy**: Advanced string matching using Levenshtein distance
- **OpenPyXL/XLRD**: Multi-format Excel file handling (.xlsx, .xls, .xlsm)
- **NumPy**: Numerical computations and array operations

### Key Algorithms Implemented

#### 1. Multi-Stage Employee Number Deduplication
```
Stage 1: Initial ID Assignment and Correction
- Identify shared employee numbers across all schedules
- Generate new unique IDs for unauthorized usage cases
- Result: 9,525 new employee numbers created

Stage 2: SSNIT and Name-Based Merging
- Group records by identical SSNIT numbers
- Apply fuzzy name matching within SSNIT groups
- Merge controller IDs where both conditions satisfied
- Result: Consolidated to 69,991 unique members

Stage 3: Trailing Zero Normalization
- Detect SSNIT numbers differing only by trailing zeros
- Merge records with similar names and normalized SSNITs
- Result: Final list of 66,866 unique members
```

#### 2. Union-Find Transitive Connection Algorithm
```
Purpose: Identify all transitively connected employee records
Implementation:
- Build graph where employee numbers are nodes
- Connect nodes appearing in same record
- Apply path compression for O(log n) performance
- Find connected components (transitive closure)

Example:
If Employee 123 appears with Employee 456
And Employee 456 appears with Employee 789
Then 123, 456, and 789 are all connected transitively
```

#### 3. Fuzzy Name Matching System
```
Configuration:
- Similarity Threshold: 80% for employee deduplication
- Similarity Threshold: 85% for SSNIT validation
- Algorithm: Token-sort ratio (Levenshtein distance)
- Requirements: Minimum 2 name parts must match

Features:
- Handles name variations (e.g., "John Smith" vs "Smith, John")
- Supports substring matching for abbreviated names
- Removes punctuation and normalizes whitespace
```

#### 4. SSNIT Classification and Validation
```
Classification Rules:
- Temporary SSNIT: Numeric only (e.g., "34619361")
- Permanent SSNIT: Letter + 12 digits (e.g., "H016310070030")
- Invalid: UNKNOWN, empty, or keyword matches

Validation Rules:
- Single person cannot have 2+ temporary SSNITs
- Single person cannot have 2+ permanent SSNITs
- Violators are split into separate person records
```

## Project Structure
```
Pension-Data-Consolidation-System/
│
├── scripts/
│   ├── part1_employee_number/
│   │   ├── 01_data_standardization.py           # Column reorganization across 1,133 files
│   │   ├── 02_master_consolidation.py           # Aggregation with duplicate detection
│   │   ├── 03_employee_deduplication.py         # Multi-stage deduplication algorithm
│   │   ├── 04_schedule_update.py                # Propagate corrections to schedules
│   │   └── 05_post_schedule_consolidation.py    # Final cleanup for trailing zeros
│   │
│   └── part2_ssnit_quality/
│       ├── 06_ssnit_clustering_test.py          # Diagnostic tool for fuzzy matching
│       ├── 07_union_find_transitive.py          # Graph-based transitive connection
│       ├── 08_ssnit_normalization.py            # Trailing zero standardization
│       ├── 09_sp_number_assignment.py           # Internal reference enrichment
│       └── 10_official_ssnit_verification.py    # Validation against government DB
│
├── docs/                                         # Detailed documentation
├── sample_data/                                  # Sample/synthetic data for testing
├── reports/                                      # Project reports and analysis
├── README.md                                     # This file
├── requirements.txt                              # Python dependencies
└── .gitignore                                   # Git ignore rules for sensitive data
```

## Script Descriptions

### Part 1: Employee Number Deduplication (Scripts 01-05)

#### Script 01: Data Standardization
**Purpose**: Normalize 1,133 heterogeneous Excel files into consistent format
**Key Features**:
- Handles multiple naming conventions for columns (EMPLOYEE_NUMBER, EE_NUMBER, etc.)
- Combines split name columns (SURNAME + OTHERNAME → FULL_NAME)
- Preserves source tracking (SOURCE_FILE, SOURCE_SHEET)
- Converts legacy .xls to .xlsx format
- Generates detailed processing logs

**Input**: Raw pension scheme files (multiple formats)
**Output**: Standardized files with consistent column structure

#### Script 02: Master Consolidation and Analysis
**Purpose**: Aggregate all standardized files into single master dataset
**Key Features**:
- Recursive directory traversal for nested folder structures
- Normalized name matching (removes commas, extra spaces)
- Duplicate detection across three dimensions (name, employee number, SSNIT)
- Data quality analysis and issue identification
- Comprehensive Excel workbook with multiple analysis sheets

**Input**: Standardized files from Script 01
**Output**: Master analysis workbook with duplicate reports

#### Script 03: Employee Number Deduplication (Core Algorithm)
**Purpose**: Resolve all employee number conflicts using multi-stage algorithm
**Key Features**:
- SSNIT classification (temporary/permanent/unknown)
- Fuzzy name grouping with 80% threshold
- SSNIT combination validation
- Transitive closure through shared SSNIT numbers
- Winner selection based on record frequency
- Sequential new ID generation from maximum existing

**Input**: Consolidated master data
**Output**: 
- Master_Data_Corrected.csv (corrected employee numbers)
- Employee_Number_Mapping.csv (old → new mapping)
- Reassigned_Employee_Numbers_Register.xlsx (reference workbook)
- Duplicate_Resolution_Report.txt (detailed report)

#### Script 04: Schedule Update and Correction
**Purpose**: Propagate corrected employee numbers back to original 2,905 schedules
**Key Features**:
- Dual-key matching (Employee Number + SSNIT) for accuracy
- Status-based updates (Reassigned vs Kept)
- Preserves empty/unknown employee numbers
- Maintains original folder structure
- Comprehensive logging (Update_Log, No_Match_Log, Error_Log)

**Input**: Master_Data_Corrected.csv + Original schedules
**Output**: Updated schedules with corrected employee numbers

#### Script 05: Post-Schedule SSNIT Consolidation
**Purpose**: Final cleanup after schedule updates to merge trailing zero duplicates
**Key Features**:
- Name normalization (lowercase, remove commas/spaces)
- SSNIT normalization (remove trailing zeros)
- Group by name, merge by normalized SSNIT
- Pipe-separated combination of employee numbers

**Input**: Master list after schedule updates
**Output**: Final consolidated master list

### Part 2: SSNIT Quality Resolution (Scripts 06-10)

#### Script 06: SSNIT Clustering Test (Diagnostic Tool)
**Purpose**: Test and validate fuzzy name matching algorithm
**Key Features**:
- Analyzes specific SSNIT numbers for clustering accuracy
- Tests name part combinations with fuzzy scoring
- Validates clustering function behavior
- Prevents false positive merges

**Input**: Schedule records for testing
**Output**: Console output with clustering analysis

#### Script 07: Union-Find Transitive Connection
**Purpose**: Merge all transitively connected employee groups
**Key Features**:
- Union-Find data structure with path compression
- Graph-based connection detection
- Efficient O(log n) lookup performance
- Collects all SSNITs and names for connected groups

**Input**: Output from fuzzy name grouping
**Output**: Fully consolidated groups with transitive connections resolved

#### Script 08: SSNIT Format Normalization
**Purpose**: Handle trailing zero anomalies in SSNIT numbers
**Key Features**:
- SSNIT normalization (removes trailing zeros)
- Detects SSNITs differing only by trailing zeros
- Groups by name, merges by normalized SSNIT
- Maintains audit trail in log file

**Input**: Output from Union-Find algorithm
**Output**: Consolidated list with 3,119 trailing zero duplicates merged

#### Script 09: SP Number Assignment
**Purpose**: Enrich records with internal pension scheme reference numbers
**Key Features**:
- Smart employee number selection (numeric priority, fewest digits)
- Fast lookup dictionary for O(1) performance
- Maps employee numbers to SP_NUMBER references

**Input**: Cleaned SSNIT data + Reference file with SP_NUMBER mappings
**Output**: Enriched file ready for official database verification

#### Script 10: Official SSNIT Database Verification
**Purpose**: Validate internal records against 54,706 official government records
**Key Features**:
- Dual matching strategy (SSNIT first, then Employee Number)
- Pipe-separated value parsing with invalid identifier filtering
- Fast O(1) lookup using dictionary-based indexing
- Fuzzy name matching (85% threshold, minimum 2 name parts)
- Identifies members using incorrect SSNIT numbers

**Input**: 
- Internal member list (enriched with SP_NUMBER)
- Official government SSNIT database (54,706 records)

**Output**: Verification report with match statistics and SP_NUMBER assignments

## Installation and Setup

### Prerequisites
```
Python 3.8 or higher
pip (Python package installer)
```

### Installation Steps

1. Clone or download this repository
2. Navigate to project directory
3. Install required dependencies:
```bash
pip install -r requirements.txt
```

### Dependencies
```
pandas==2.0.3          # Data manipulation and analysis
openpyxl==3.1.2        # Excel file handling (.xlsx)
xlrd==2.0.1            # Legacy Excel file handling (.xls)
fuzzywuzzy==0.18.0     # Fuzzy string matching
python-Levenshtein==0.21.1  # Fast string distance calculations
numpy==1.24.3          # Numerical computations
```

## Usage Instructions

### Configuration

Before running any script, update the configuration section at the top of each file:
```python
# UPDATE THESE PATHS FOR YOUR ENVIRONMENT
INPUT_FILE = r"C:\path\to\your\input_file.csv"
OUTPUT_FOLDER = r"C:\path\to\your\output_folder"
```

### Execution Order

**Part 1: Employee Number Deduplication**
```bash
# Step 1: Standardize all source files
python scripts/part1_employee_number/01_data_standardization.py

# Step 2: Consolidate into master dataset
python scripts/part1_employee_number/02_master_consolidation.py

# Step 3: Generate new employee numbers for conflicts
python scripts/part1_employee_number/03_employee_deduplication.py

# Step 4: Update all schedules with corrected numbers
python scripts/part1_employee_number/04_schedule_update.py

# Step 5: Final consolidation after schedule updates
python scripts/part1_employee_number/05_post_schedule_consolidation.py
```

**Part 2: SSNIT Quality Resolution**
```bash
# Step 6 (Optional): Test clustering algorithm
python scripts/part2_ssnit_quality/06_ssnit_clustering_test.py

# Step 7: Apply Union-Find transitive connection
python scripts/part2_ssnit_quality/07_union_find_transitive.py

# Step 8: Normalize SSNIT format (trailing zeros)
python scripts/part2_ssnit_quality/08_ssnit_normalization.py

# Step 9: Assign internal SP reference numbers
python scripts/part2_ssnit_quality/09_sp_number_assignment.py

# Step 10: Verify against official government database
python scripts/part2_ssnit_quality/10_official_ssnit_verification.py
```

## Output Files Generated

### Part 1 Outputs
- `Master_Data_Corrected.csv` - Complete dataset with corrected employee numbers
- `Employee_Number_Mapping.csv` - Mapping table (old → new employee numbers)
- `Reassigned_Employee_Numbers_Register.xlsx` - Reference workbook with multiple sheets
- `Duplicate_Resolution_Report.txt` - Detailed processing report
- `Records_For_Manual_Review.csv` - Flagged cases requiring human verification
- `Update_Log.csv` - Every change made to schedules
- `Schedule_Update_Report.txt` - Summary of schedule updates

### Part 2 Outputs
- Fully consolidated member list with transitive connections resolved
- SSNIT normalization log showing trailing zero merges
- SP_NUMBER enriched dataset
- Official database verification report with match statistics
- Lists of missing SSNIT members
- Lists of members using incorrect SSNIT numbers

## Performance Metrics

### Processing Statistics
- **Total Records Processed**: 3,765,847 contribution records
- **Source Files Handled**: 1,133 Excel files
- **Monthly Schedules Generated**: 2,905 standardized schedules
- **Processing Time**: Complete workflow executes in under 2 hours on standard hardware

### Deduplication Efficiency
- **Initial Employee Numbers**: 100,548 unique identifiers
- **After Stage 1**: 100,548 (9,525 new IDs generated)
- **After Stage 2**: 69,991 (SSNIT/name merging)
- **After Stage 3**: 66,866 (trailing zero normalization)
- **Final SSNIT Resolution**: 66,719 unique validated members

### Data Quality Improvement
- **Records Ready for Processing**: 44,932 members (67.4%)
- **Missing SSNIT Identified**: 1,253 members requiring registration
- **Incorrect SSNIT Detected**: 111 members using wrong numbers
- **Duplicate Accounts Prevented**: Systematic identification of potential duplicate benefit payments

## Technical Highlights for Recruiters

### Advanced Programming Techniques
- **Graph Algorithms**: Union-Find with path compression for transitive closure
- **Fuzzy Matching**: Levenshtein distance with token-sort ratio for name variations
- **ETL Pipeline**: Multi-stage data transformation with comprehensive error handling
- **Performance Optimization**: Dictionary-based O(1) lookups processing millions of records
- **Data Validation**: Multi-layered validation against business rules and official databases

### Software Engineering Best Practices
- **Modular Design**: 10 independent scripts with clear separation of concerns
- **Comprehensive Logging**: Detailed audit trails for every transformation
- **Error Handling**: Robust exception handling with graceful degradation
- **Documentation**: Inline comments and comprehensive README
- **Version Control**: Git-ready structure with proper .gitignore

### Business Impact
- **Eliminated Manual Processing**: Reduced 4-hour manual tasks to 5-minute automated processes
- **Ensured Payment Accuracy**: Prevented duplicate benefit payments through systematic duplicate detection
- **Regulatory Compliance**: Validated 100% of records against official government database
- **Operational Efficiency**: Enabled processing of 67.4% of members immediately

## Data Privacy and Security

### Important Notice
This repository contains **NO actual pension data**. All sensitive information has been removed for privacy and security compliance. Scripts are provided for methodology demonstration and portfolio purposes only.

### Sensitive Data Exclusions
The `.gitignore` file is configured to prevent uploading:
- All Excel files (.xlsx, .xls, .xlsm)
- All CSV files (.csv)
- All output folders and logs
- Any folders containing source data

## Future Enhancements

### Potential Improvements
1. **Machine Learning Integration**: Train models to predict data quality issues
2. **Real-time Processing**: Convert batch scripts to streaming pipeline
3. **Web Interface**: Build dashboard for non-technical users
4. **API Development**: Expose deduplication algorithms as REST API
5. **Automated Testing**: Implement unit tests for critical functions
6. **Database Integration**: Direct database connections instead of Excel files

### Scalability Considerations
- Current implementation handles millions of records efficiently
- For 10M+ records, consider distributed processing (Spark/Dask)
- For production deployment, implement database backend

## Lessons Learned

### Technical Learnings
- Fuzzy matching thresholds require domain expertise and iterative tuning
- Graph algorithms (Union-Find) are essential for complex relationship resolution
- Comprehensive logging is critical for troubleshooting large-scale data operations
- Performance optimization through dictionary lookups crucial for multi-million record datasets

### Business Insights
- Data quality issues often stem from systemic process problems, not just data entry errors
- Automated validation against authoritative sources (government databases) is essential
- Human verification still required for edge cases despite sophisticated algorithms
- Clear audit trails build stakeholder confidence in automated processes

## About the Author

**Andy Baiden**  
Data Analyst & Automation Specialist  
Accra, Ghana

**Contact Information**:
- Email: abaiden514@gmail.com
- Phone: +233 243144001
- GitHub: [github.com/YOUR_USERNAME]

**Professional Background**:
Results-driven Data Analyst specializing in building custom automation tools to solve complex business problems. Expert in transforming inefficient manual processes into streamlined automated systems using Python, advanced Excel/VBA, and modern data science techniques.

**Current Role**: Data Analyst at Standard Pensions Trust (January 2025 - Present)

## Project Timeline

- **Project Start**: January 2025
- **Part 1 Completion**: February 2025
- **Part 2 Completion**: March 2025
- **Documentation**: Ongoing

## Acknowledgments

Project completed as part of data quality improvement initiative at Standard Pensions Trust, Ghana. Special recognition to the Operations and IT teams for their collaboration and domain expertise.

## License

This project is provided for portfolio demonstration purposes. The methodology and algorithms are available for reference, but the specific business logic remains proprietary to the implementing organization.

---

**Last Updated**: December 2025  
**Version**: 1.0  
**Status**: Production-Ready