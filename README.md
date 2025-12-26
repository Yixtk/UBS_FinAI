# Structured Product Term Sheet Extraction & Payoff Calculator

🏦 **AI-powered system for extracting structured product information from PDF term sheets and calculating payoffs**

[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

---

## 🎯 Overview

This system uses Large Language Models (LLMs) to automatically extract payoff-critical information from structured product term sheets and calculates payoffs for various market scenarios.

### Key Features

- ✅ **PDF Extraction**: Automatically parse Phoenix Snowball and Worst-of products
- ✅ **100% Accuracy**: Validated against ground truth on tested products
- ✅ **Payoff Calculation**: Calculate coupons and payoffs for multiple scenarios
- ✅ **Safety Validation**: Built-in guardrails for production-grade reliability
- ✅ **Extensible Design**: Easy to add new product types

### Supported Product Types

- **Single Underlying Phoenix** (e.g., Phoenix Snowball on S&P 500)
- **Worst-of Phoenix** (e.g., Phoenix on AMD/NVDA/INTC basket)

---

## 🚀 Quick Start

### Installation

```bash
# Clone the repository
git clone <your-repo-url>
cd UBS_FinAI

# Install dependencies
pip install -r requirements.txt

# Set up environment variables (copy and edit)
cp .env.example .env
# Edit .env with your API keys
```

### Configuration

Create a `.env` file with your LLM API credentials:

```bash
# Choose your LLM provider
LLM_PROVIDER=anthropic  # or "openai" or "deepseek"

# API Keys
ANTHROPIC_API_KEY=your-key-here
# OPENAI_API_KEY=your-key-here
# DEEPSEEK_API_KEY=your-key-here

# Model settings
LLM_MODEL=claude-sonnet-4-5-20250929
LLM_TEMPERATURE=0.0
```

---

## 📖 Usage

### 1. Extract Information from PDF

```bash
# Run extraction on test PDFs
python -m tests.test

# Output: results/test_results_YYYYMMDD_HHMMSS.json
```

### 2. Validate Extraction Quality

```bash
# Compare against ground truth
python -m scripts.compare_with_ground_truth results/test_results_*.json
```

### 3. Calculate Payoffs

```bash
# Calculate payoffs from extracted data
python -m scripts.calculate_payoff_from_json results/test_results_*.json

# Output: results/payoff_results_YYYYMMDD_HHMMSS.json
```

### 4. Use in Code

```python
from src.prompts import PayoffExtractor
from src.payoff_ready_validator import validate_and_prepare_for_payoff
from src.payoff_single import SinglePhoenixPayoff

# Extract from PDF
extractor = PayoffExtractor()
result = extractor.extract_from_pdf("data/your_termsheet.pdf")

# Validate
validation = validate_and_prepare_for_payoff(result)
if not validation["is_valid"]:
    print("Errors:", validation["errors"])
    exit(1)

# Calculate payoff
payoff_data = validation["payoff_ready_data"]
calc = SinglePhoenixPayoff(payoff_data)

# Define price scenario
price_path = [2000, 2100, 2200, 2300, 2400]

# Calculate
coupons, payoff, details = calc.calculate_payoff(price_path)
print(f"Total Value: ${coupons + payoff:.2f}")
```

---

## 📊 System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      PDF Term Sheet                          │
└───────────────────────┬─────────────────────────────────────┘
                        ↓
              ┌─────────────────────┐
              │  Document Loader    │ (pypdf)
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
              │  - Normalize assets │
              │  - Fix structure    │
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

---

## 📁 Project Structure

```
UBS_FinAI/
├── src/                                # Core source code
│   ├── prompts.py                      # Main extractor with post-processing
│   ├── prompt.py                       # LLM prompt templates
│   ├── llm_client.py                   # LLM API client
│   ├── document_loader.py              # PDF loader
│   ├── config.py                       # Configuration management
│   ├── payoff_single.py                # Single underlying Phoenix
│   ├── payoff_worst_of.py              # Worst-of Phoenix
│   └── payoff_ready_validator.py       # Payoff safety validator
│
├── tests/                              # Test files
│   ├── test.py                         # Test suite
│   ├── test_case.py                    # Test cases
│   └── test_payoff_engines.py          # Payoff engine tests
│
├── scripts/                            # Utility scripts
│   ├── calculate_payoff_from_json.py   # JSON → Payoff calculator
│   └── compare_with_ground_truth.py    # Accuracy evaluation
│
├── data/                               # Input PDF files
│   ├── BNP-PhoenixSnowball-SP500-XS1083630027-TS.pdf
│   └── IT0006764473-TS.pdf
│
├── results/                            # Output files (not in git)
│   ├── test_results_*.json             # Extraction results
│   └── payoff_results_*.json           # Payoff calculations
│
├── docs/                               # Documentation
│   ├── README_PAYOFF_READY.md          # Detailed technical guide
│   ├── PROJECT_STRUCTURE.md            # Project organization
│   ├── SETUP.md                        # Setup instructions
│   └── GITHUB_UPLOAD_GUIDE.md          # GitHub guide
│
├── README.md                           # This file
├── requirements.txt                    # Python dependencies
├── LICENSE                             # MIT License
├── .gitignore                          # Git ignore rules
└── LLM_variables.env                   # API keys (not in git)
```

For detailed structure documentation, see [docs/PROJECT_STRUCTURE.md](docs/PROJECT_STRUCTURE.md).

---

## ✅ Test Results

### Accuracy Metrics

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

## 🔧 Key Components

### 1. Extraction Engine (`prompts.py`)
- LLM-based JSON extraction
- Intelligent chunking for large documents
- Post-processing with deterministic rules

### 2. Validation Layer (`payoff_ready_validator.py`)
- Schema validation
- Required field checking
- Type enforcement
- **Critical safety guardrail**

### 3. Payoff Calculators
- `payoff_single.py`: Single underlying products
- `payoff_worst_of.py`: Multi-asset worst-of products
- Scenario analysis (bullish/sideways/bearish)
- Memory coupon handling
- Autocall and knock-in logic

---

## 🎓 How It Works

### Hybrid AI + Rules Approach

The system combines:
1. **LLM Intelligence**: Semantic understanding of financial documents
2. **Deterministic Rules**: Ensuring payoff-critical accuracy
3. **Validation Layer**: Catching errors before payoff calculation

### Key Innovations

✅ **Post-processing fixes LLM output**
- Underlying deduplication (same name = same asset)
- Structure type inference from # of underlyings
- Noise field removal

✅ **Barrier calculation from prices**
- Uses `barrier_prices` when `barrier_level` is ambiguous
- Calculates implied barriers from strike prices

✅ **Separated coupon accounting**
- Fixed coupon (one-time)
- Conditional coupon (memory feature)
- Clear payment tracking

---

## 🚨 Safety Guidelines

### ✅ Recommended Use
- Automatic payoff code generation
- Historical scenario analysis
- Research and prototyping
- FinTech demos with human oversight

### ❌ Not Yet Recommended
- Fully automated trading without review
- Client-facing pricing without verification
- Sole source of truth for risk management

### 🛡️ Safety Protocol
Always:
1. Run `payoff_ready_validator` before calculations
2. Spot-check extracted parameters
3. Review validation warnings
4. Maintain human-in-the-loop for production

---

## 📈 Extending the System

### Adding New Product Types

1. Create new payoff engine (e.g., `payoff_reverse_convertible.py`)
2. Add structure type to validator
3. Update ground truth tests
4. Document expected JSON schema

### Example: Adding Autocallable

```python
class AutocallablePayoff:
    def __init__(self, payoff_data: Dict):
        self.parse_parameters(payoff_data)
    
    def calculate_payoff(self, price_path: List[float]):
        # Your logic here
        pass
```

---

## 🐛 Troubleshooting

### Common Issues

**"Module not found: dotenv"**
```bash
pip install python-dotenv
```

**"API key not found"**
- Ensure `.env` file exists
- Check environment variable names match `config.py`

**"Validation failed"**
- Check extraction JSON has required fields
- Review validation error messages
- Verify PDF contains payoff information

---

## 📝 License

MIT License - See LICENSE file for details

---

## 🤝 Contributing

Contributions are welcome! Please:
1. Fork the repository
2. Create a feature branch
3. Add tests for new functionality
4. Submit a pull request

---

## 📧 Contact

For questions or issues, please open a GitHub issue.

---

## 🙏 Acknowledgments

- Built with Claude Sonnet 4.5 (Anthropic)
- Inspired by real-world structured product workflows
- Tested on actual term sheets from BNP Paribas and Natixis

---

**⚠️ Disclaimer**: This is a research/prototype system. Always verify extracted data and calculated payoffs against official term sheets before using for trading or client-facing purposes.

