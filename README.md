# Structured Product Payoff Extraction & Calculator

🏦 **AI-powered system for extracting structured product information from PDF term sheets and calculating payoffs**

[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

---

## 🎯 Overview

This system uses Large Language Models to automatically extract payoff information from structured product term sheets and calculates payoffs for various market scenarios.

### Key Features

- ✅ **Automated PDF Extraction**: Parse Phoenix Snowball and Worst-of products
- ✅ **High Accuracy**: Validated against ground truth with deterministic post-processing
- ✅ **Payoff Calculation**: Calculate coupons and payoffs for multiple scenarios
- ✅ **Safety Validation**: Built-in guardrails for production-grade reliability
- ✅ **Extensible**: Easy to add new product types

### Supported Products

- **Single Underlying Phoenix** (e.g., Phoenix Snowball on S&P 500)
- **Worst-of Phoenix** (e.g., Phoenix on AMD/NVDA/INTC basket)

---

## 🚀 Quick Start

### Installation

```bash
git clone https://github.com/Yixtk/UBS_FinAI.git
cd UBS_FinAI
pip install -r requirements.txt
```

### Configuration

Create `LLM_variables.env` with your API key:

```bash
# Choose your LLM provider
LLM_PROVIDER=anthropic  # or "openai" / "deepseek"
ANTHROPIC_API_KEY=your-api-key-here
LLM_MODEL=claude-3-sonnet-20240229
LLM_TEMPERATURE=0.0
```

### Usage

```bash
# 1. Extract from PDF
python -m tests.test

# 2. Calculate payoffs
python -m scripts.calculate_payoff_from_json results/test_results_*.json

# 3. Compare with ground truth
python -m scripts.compare_with_ground_truth results/test_results_*.json
```

---

## 📖 Example

```python
from src.prompts import PayoffExtractor
from src.payoff_single import SinglePhoenixPayoff

# Extract from PDF
extractor = PayoffExtractor()
result = extractor.extract_from_pdf("data/your_termsheet.pdf")

# Calculate payoff
calc = SinglePhoenixPayoff(result)
price_path = [2000, 2100, 2200, 2300, 2400]
coupons, payoff, details = calc.calculate_payoff(price_path)

print(f"Total Value: ${coupons + payoff:.2f}")
```

---

## 📁 Project Structure

```
UBS_FinAI/
│
├── src/                              # 🔧 Core source code
│   ├── config.py                     # Configuration and API key management
│   ├── llm_client.py                 # Unified LLM API client (OpenAI/Anthropic/DeepSeek)
│   ├── document_loader.py            # PDF text extraction utilities
│   ├── prompt.py                     # LLM prompt templates
│   ├── prompts.py                    # PayoffExtractor - main extraction orchestrator
│   ├── payoff_ready_validator.py     # Schema validation before payoff calculation
│   ├── payoff_single.py              # Single underlying Phoenix payoff engine
│   └── payoff_worst_of.py            # Worst-of Phoenix payoff engine
│
├── tests/                            # 🧪 Test files
│   ├── test.py                       # Main extraction test suite
│   ├── test_case.py                  # Test case definitions with expected outcomes
│   └── test_payoff_engines.py        # Integration tests for payoff engines
│
├── scripts/                          # 🛠️ Utility scripts
│   ├── calculate_payoff_from_json.py # Calculate payoffs from extraction JSON
│   └── compare_with_ground_truth.py  # Accuracy evaluation against ground truth
│
├── data/                             # 📄 Input data
│   ├── BNP-PhoenixSnowball-SP500-XS1083630027-TS.pdf
│   └── IT0006764473-TS.pdf
│
├── results/                          # 📊 Output files (not in git)
│   ├── test_results_*.json           # Extraction results with timestamps
│   ├── payoff_results_*.json         # Payoff calculation results
│   └── *_comparison.json             # Ground truth comparison reports
│
├── docs/                             # 📚 Documentation
│   ├── README_PAYOFF_READY.md        # Detailed payoff system documentation
│   ├── PROJECT_STRUCTURE.md          # Complete project structure guide
│   ├── SETUP.md                      # Setup and installation instructions
│   ├── GITHUB_UPLOAD_GUIDE.md        # GitHub upload guide
│   ├── BNP Phoenix Snowball analysis.pdf
│   ├── term_sheet_extraction.pdf
│   └── termsheet search keywords.docx
│
├── README.md                         # This file
├── requirements.txt                  # Python dependencies
├── LICENSE                           # MIT License
├── .gitignore                        # Git ignore rules
└── LLM_variables.env                 # API keys configuration (not in git)
```

### Core Module Descriptions

#### `src/` - Core Source Code

| File | Description |
|------|-------------|
| **config.py** | Loads API keys from `LLM_variables.env`, manages LLM provider settings |
| **llm_client.py** | Unified client for OpenAI, Anthropic, DeepSeek APIs with automatic retry |
| **document_loader.py** | PDF text extraction using `pypdf`, text chunking for LLM processing |
| **prompt.py** | LLM prompt templates for extraction, validation, and section parsing |
| **prompts.py** | `PayoffExtractor` class - orchestrates extraction with post-processing rules |
| **payoff_ready_validator.py** | Safety validation: schema checks, required fields, type enforcement |
| **payoff_single.py** | Payoff calculator for single-underlying Phoenix products |
| **payoff_worst_of.py** | Payoff calculator for worst-of Phoenix products with memory coupons |

#### `tests/` - Test Suite

| File | Description |
|------|-------------|
| **test.py** | Runs extraction on test PDFs, validates schema, saves results to JSON |
| **test_case.py** | Defines test cases with expected structure types and required fields |
| **test_payoff_engines.py** | Integration tests with simulated market scenarios (bullish/bearish/sideways) |

#### `scripts/` - Utility Scripts

| File | Description |
|------|-------------|
| **calculate_payoff_from_json.py** | End-to-end: reads extraction JSON → validates → calculates payoffs → saves results |
| **compare_with_ground_truth.py** | Compares AI extraction against human-verified ground truth, generates accuracy report |

---

## 🔧 System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      PDF Term Sheet                          │
└───────────────────────┬─────────────────────────────────────┘
                        ↓
              ┌─────────────────────┐
              │  Document Loader    │ (pypdf)
              │  - Extract text     │
              │  - Page splitting   │
              └──────────┬──────────┘
                         ↓
              ┌─────────────────────┐
              │  PayoffExtractor    │ (LLM + Prompt)
              │  - Chunk processing │
              │  - JSON extraction  │
              └──────────┬──────────┘
                         ↓
              ┌─────────────────────┐
              │  Post-Processor     │ ✅ Deterministic Rules
              │  - Deduplicate      │
              │  - Normalize type   │
              │  - Clean noise      │
              └──────────┬──────────┘
                         ↓
              ┌─────────────────────┐
              │  Payoff Validator   │ ✅ Safety Guardrail
              │  - Schema check     │
              │  - Required fields  │
              └──────────┬──────────┘
                         ↓
              ┌─────────────────────┐
              │  Payoff Engine      │
              │  - Single Phoenix   │
              │  - Worst-of Phoenix │
              └─────────────────────┘
```

### Key Innovations

✅ **Hybrid AI + Rules**: LLM semantic understanding + deterministic post-processing  
✅ **Underlying Deduplication**: Same name = same asset, merge fields  
✅ **Structure Type Inference**: Automatically determine single/worst-of from # of underlyings  
✅ **Barrier Calculation**: Uses `barrier_prices` when `barrier_level` is ambiguous  
✅ **Separated Coupon Accounting**: Fixed vs. conditional coupons clearly tracked

---

## ✅ Accuracy Metrics

Tested on real term sheets from BNP Paribas and Natixis:

| Metric | Score |
|--------|-------|
| Structure Type Classification | 100% (2/2) |
| Underlying Extraction | 100% (4/4) |
| Date Extraction | 100% (49/49) |
| Payoff Component Coverage | 100% |
| **Overall Payoff-Ready** | **100%** |

### Tested Products

1. **BNP Phoenix Snowball on S&P 500**
   - Structure: Single underlying
   - Features: Conditional coupon, autocall, knock-in

2. **Natixis Phoenix on AMD/NVDA/INTC**
   - Structure: Worst-of
   - Features: Fixed coupon, memory coupon, autocall

---

## 📚 Documentation

For more detailed information:

- **[Setup Guide](docs/SETUP.md)** - Installation, configuration, and troubleshooting
- **[Project Structure](docs/PROJECT_STRUCTURE.md)** - Detailed module documentation and data flow
- **[Payoff System](docs/README_PAYOFF_READY.md)** - Technical details on payoff calculation
- **[GitHub Guide](docs/GITHUB_UPLOAD_GUIDE.md)** - Contribution and deployment guide

---

## 🚨 Safety & Disclaimer

### ✅ Recommended Use
- Automatic payoff code generation with human review
- Historical scenario analysis
- Research and prototyping
- FinTech demos with oversight

### ❌ Not Yet Recommended
- Fully automated trading without verification
- Client-facing pricing without spot-checks
- Sole source of truth for risk management

### 🛡️ Safety Protocol
Always:
1. Run `payoff_ready_validator` before calculations
2. Spot-check extracted parameters against term sheets
3. Review validation warnings
4. Maintain human-in-the-loop for production

⚠️ **This is a research/prototype system.** Always verify extracted data and calculated payoffs against official term sheets.

---

## 🙏 Acknowledgments

This project builds upon and extends the original work from:

### Original Framework
- **[gdgdandsz/UBS_termsheet](https://github.com/gdgdandsz/UBS_termsheet)** - Original term sheet extraction framework  
  Special thanks for the foundational extraction architecture and LLM integration approach.

### Team Contributions
- **BNP Phoenix Snowball Analysis** (docs/) - Detailed product structure analysis
- **Term Sheet Extraction Research** (docs/) - Methodology and keyword research
- **Team Members' Financial Analysis** - Product structure validation and testing

### Enhancements in This Version

- ✨ Organized project structure with `src/`, `tests/`, `scripts/` separation
- ✨ Dedicated payoff calculation engines for single and worst-of products
- ✨ Post-processing layer with deterministic rules for data normalization
- ✨ Payoff validation system with schema checking and safety guardrails
- ✨ Ground truth comparison and accuracy evaluation framework
- ✨ Comprehensive documentation and usage examples
- ✨ Support for multiple LLM providers (OpenAI, Anthropic, DeepSeek)
- ✨ Separated fixed/conditional coupon accounting
- ✨ Barrier calculation from prices when levels are ambiguous

---

## 📝 License

MIT License - See [LICENSE](LICENSE) file for details.

---

## 📧 Contact

For questions or issues, please open a [GitHub issue](https://github.com/Yixtk/UBS_FinAI/issues).

---

**Built with Claude Sonnet 4.5** | **Tested on actual structured product term sheets**
