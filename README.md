# Threat Modeler CLI

A comprehensive CLI tool for automated threat modeling assessments based on Confluence documentation. This tool leverages Large Language Models (LLMs) to analyze system documentation and generate detailed threat models with iterative gap analysis.

## Features

- **Confluence Integration**: Automatically fetch documentation and images from Confluence pages
- **LLM Support**: Compatible with OpenAI and Azure OpenAI with custom URLs and API versions
- **Iterative Analysis**: Automated gap analysis with up to 10 iterations for model improvement
- **Flexible Output**: Markdown-formatted threat models with timestamped directories
- **Configurable Prompts**: External prompt files for easy customization
- **Rich CLI**: Beautiful console interface with progress indicators
- **Advanced Image Support**: Downloads and includes images from Confluence page attachments via REST API in LLM analysis, with automatic SVG to PNG conversion for LLM compatibility

## Installation

### Prerequisites

- Python 3.8 or higher
- Confluence API access (username and token)
- OpenAI API key
- **Optional for SVG support**: Cairo graphics library (see SVG Support section below)

### Install from Source

```bash
git clone https://github.com/your-org/threat-modeler.git
cd threat-modeler
pip install -e .
```

### Install with pip

```bash
pip install threat-modeler
```

### SVG Support (Optional)

The tool automatically converts SVG images to PNG format for LLM compatibility. This requires the Cairo graphics library:

**macOS (using Homebrew):**
```bash
brew install cairo
pip install cairosvg
```

**Ubuntu/Debian:**
```bash
sudo apt-get install libcairo2-dev
pip install cairosvg
```

**CentOS/RHEL/Fedora:**
```bash
# CentOS/RHEL
sudo yum install cairo-devel
# Fedora
sudo dnf install cairo-devel

pip install cairosvg
```

**Note**: If Cairo is not installed, SVG images will be skipped with a warning, but all other functionality works normally.

## Quick Start

1. **Initialize the project** (creates default config files):
```bash
threat-modeler init
```

2. **Set environment variables** (recommended):
```bash
export CONFLUENCE_USERNAME="your_username"
export CONFLUENCE_TOKEN="your_api_token"
export OPENAI_API_KEY="your_openai_api_key"
```

3. **Run threat modeling assessment**:
```bash
threat-modeler run --confluence-url "https://your-domain.atlassian.net/wiki/spaces/SPACE/pages/PAGEID"
```

## Configuration

### Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `CONFLUENCE_USERNAME` | Your Confluence username | Yes |
| `CONFLUENCE_TOKEN` | Your Confluence API token | Yes |
| `OPENAI_API_KEY` | Your OpenAI API key | Yes* |

*Required when using OpenAI provider

### Configuration Files

The tool uses YAML configuration files. After running `threat-modeler init`, you can customize:

- **`configs/default.yaml`**: Main configuration settings
- **`prompts/threat_model_generation.txt`**: Prompt for initial threat model generation
- **`prompts/gap_analysis.txt`**: Prompt for gap analysis

### CLI Options

```bash
threat-modeler run --help
```

#### Main Options

- `--confluence-url`: Confluence page URL to analyze (required)
- `--confluence-username`: Confluence username (overrides env var)
- `--confluence-token`: Confluence API token (overrides env var)
- `--llm-provider`: LLM provider (default: openai)
- `--llm-model`: LLM model (default: gpt-4)
- `--openai-api-key`: OpenAI API key (overrides env var)
- `--azure-api-key`: Azure OpenAI API key (overrides env var)
- `--azure-base-url`: Azure OpenAI base URL (e.g., https://your-resource.openai.azure.com/)
- `--azure-deployment`: Azure OpenAI deployment name
- `--azure-api-version`: Azure OpenAI API version (default: 2023-12-01-preview)
- `--azure-auth-method`: Azure authentication method: api_key, aad, managed_identity (default: api_key)
- `--output-dir`: Output directory (default: ./output)
- `--max-iterations`: Maximum gap analysis iterations (default: 10)
- `--generate-diagram`: Generate DFDv3 diagram in drawio format after threat model completion
- `--verbose`: Enable verbose logging

## Usage Examples

### Basic Usage

```bash
# Using environment variables for credentials
threat-modeler run \
  --confluence-url "https://company.atlassian.net/wiki/spaces/PROJ/pages/123456789/Documentation"

# Specifying credentials explicitly
threat-modeler run \
  --confluence-url "https://company.atlassian.net/wiki/spaces/PROJ/pages/123456789/Documentation" \
  --confluence-username "john.doe" \
  --confluence-token "ATATT3xFfGF0EJqLmEJSJ4JzUwQ8zKzMw" \
  --openai-api-key "sk-..."
```

### Advanced Usage

```bash
# Custom output directory with verbose logging
threat-modeler run \
  --confluence-url "https://company.atlassian.net/wiki/spaces/PROJ/pages/123456789/Architecture" \
  --output-dir "./threat-models" \
  --max-iterations 5 \
  --verbose

# Using different LLM model
threat-modeler run \
  --confluence-url "https://company.atlassian.net/wiki/spaces/PROJ/pages/123456789/Design" \
  --llm-model "gpt-3.5-turbo" \
  --max-iterations 3

## Azure OpenAI Authentication Methods

The Threat Modeler CLI supports three Azure OpenAI authentication methods:

### 1. **API Key Authentication** (Default)
Most common method using Azure OpenAI API keys.

```bash
threat-modeler run \
  --confluence-url "https://company.atlassian.net/wiki/spaces/PROJ/pages/123456789/Documentation" \
  --azure-api-key "your-azure-api-key" \
  --azure-base-url "https://your-resource.openai.azure.com/" \
  --azure-deployment "your-gpt4-deployment" \
  --azure-auth-method "api_key" \
  --generate-diagram
```

### 2. **Azure Active Directory (AAD) Authentication**
Uses Azure AD tokens - no API key required. Requires `azure-identity` package.

```bash
# Install azure-identity for AAD authentication
pip install azure-identity

threat-modeler run \
  --confluence-url "https://company.atlassian.net/wiki/spaces/PROJ/pages/123456789/Documentation" \
  --azure-base-url "https://your-resource.openai.azure.com/" \
  --azure-deployment "your-gpt4-deployment" \
  --azure-auth-method "aad"
```

### 3. **Managed Identity Authentication**
Uses Azure Managed Identity - ideal for Azure resources. Requires `azure-identity` package.

```bash
# Install azure-identity for Managed Identity authentication
pip install azure-identity

threat-modeler run \
  --confluence-url "https://company.atlassian.net/wiki/spaces/PROJ/pages/123456789/Documentation" \
  --azure-base-url "https://your-resource.openai.azure.com/" \
  --azure-deployment "your-gpt4-deployment" \
  --azure-auth-method "managed_identity"
```

### Environment Variables for Azure

```bash
export AZURE_OPENAI_API_KEY="your-azure-api-key"          # For API key auth
export AZURE_OPENAI_BASE_URL="https://your-resource.openai.azure.com/"
export AZURE_OPENAI_DEPLOYMENT="your-gpt4-deployment"
export AZURE_OPENAI_API_VERSION="2024-12-01-preview"      # Configurable API version
export AZURE_OPENAI_AUTH_METHOD="api_key"                 # api_key, aad, or managed_identity
```

## Enhanced DFDv3 Diagram Generation with Gap Analysis

The Threat Modeler CLI generates high-quality Data Flow Diagrams (DFDv3) using an **iterative gap analysis process** that ensures comprehensive coverage of all security-relevant components.

### 🎯 **Iterative Improvement Process:**
1. **Initial Generation** - Create first diagram from threat model
2. **Gap Analysis** - LLM analyzes diagram completeness (5 areas)
3. **Quality Assessment** - Rate completeness on 1-10 scale
4. **Iterative Improvement** - Up to 5 iterations to address gaps
5. **Final Output** - Professional drawio diagram

### 🔍 **Gap Analysis Areas:**

The diagram generation process analyzes **5 critical areas**:

1. **🎯 Missing Components**
   - External entities, processes, data stores
   - Trust boundaries and security controls

2. **🔄 Data Flow Completeness**
   - All connections between components
   - Trust boundary crossings
   - Security control visualization

3. **🔒 Security Aspects**
   - Security-critical components highlighted
   - Trust boundaries clearly marked
   - Sensitive data flows identified

4. **📐 Layout and Clarity**
   - Logical organization
   - Clear connections and labels
   - Sufficient security details

5. **🎯 Threat Model Alignment**
   - Accurate representation of threat model
   - All assets and entry points visualized
   - Security controls properly represented

### Diagram Features

- **DFDv3 Standard**: Follows Data Flow Diagram v3 specification
- **Iterative Quality Assurance**: Up to 5 improvement iterations
- **Comprehensive Coverage**: All security-relevant components included
- **Visual Elements**:
  - 🔴 **Trust Boundaries** (red ellipses)
  - 🔵 **External Entities** (gray ellipses)
  - □ **Processes** (white rectangles)
  - ⬟ **Data Stores** (gray parallelograms)
  - → **Data Flows** (arrows with labels)
- **Quality Scoring**: 1-10 completeness rating
- **Legend**: Included for easy interpretation
- **Drawio Format**: Compatible with diagrams.net/draw.io

### Usage

```bash
# Generate comprehensive threat model WITH enhanced diagram
threat-modeler run \
  --confluence-url "https://your-domain.atlassian.net/wiki/spaces/SPACE/pages/PAGEID" \
  --azure-api-key "your-key" \
  --azure-base-url "https://your-resource.openai.azure.com/" \
  --azure-deployment "gpt-4-deployment" \
  --generate-diagram \
  --verbose

# Expected output during execution:
# ✅ Threat modeling assessment completed successfully!
# ✅ Generating DFDv3 diagram...
# ✅ Diagram analysis iteration 1/5
# ✅ Diagram analysis iteration 2/5 (if gaps found)
# ✅ Diagram is complete - stopping iterations
# ✅ DFDv3 diagram saved to: /path/to/output/threat_model_diagram_Title.drawio

# Output files created:
# 📄 threat_model_Your_Page_Title.md (comprehensive threat model)
# 🎨 threat_model_diagram_Your_Page_Title.drawio (enhanced visual diagram)
```

### 🔄 **What Happens During Diagram Generation:**

```bash
# Phase 1: Initial Diagram Generation
INFO - Generating DFDv3 diagram...
INFO - LLM connection validated

# Phase 2: Quality Assurance Loop (up to 5 iterations)
INFO - Diagram analysis iteration 1/5
INFO - Diagram analysis iteration 2/5  # Only if gaps found
INFO - Diagram is complete - stopping iterations

# Phase 3: Final Output
INFO - DFDv3 diagram saved to: /path/to/output/threat_model_diagram_Title.drawio
```

### Diagram Content

The generated diagram includes:

1. **Page Title**: Based on Confluence page title
2. **Trust Boundary**: Security perimeter around the system
3. **External Entities**: Users, external systems, APIs
4. **Processes**: Main application components and services
5. **Data Stores**: Databases, file storage, external services
6. **Data Flows**: Connections and data movement between components
7. **Legend**: Visual guide for diagram elements

### Opening Diagrams

1. **Online**: Go to [app.diagrams.net](https://app.diagrams.net)
2. **Upload**: Import the `.drawio` file
3. **Edit**: Modify the diagram as needed
4. **Export**: Save as PNG, PDF, or other formats

### Example Output Structure

```
output/
└── threat_model_20241227_143052/
    ├── threat_model_Your_Page_Title.md
    └── threat_model_diagram_Your_Page_Title.drawio
```
### Configuration Customization

```bash
# Edit the default configuration
vim configs/default.yaml

# Customize prompts
vim prompts/threat_model_generation.txt
vim prompts/gap_analysis.txt
```

## How It Works

1. **Document Fetching**: Extracts content and images from Confluence pages
2. **Initial Analysis**: Uses LLM to generate comprehensive threat model
3. **Gap Analysis**: Iteratively analyzes and improves the threat model
4. **Output Generation**: Creates timestamped Markdown files with complete analysis

### Process Flow

```
Confluence Page → LLM Analysis → Threat Model Generation
                                       ↓
Gap Analysis Loop (max 10 iterations) → Final Threat Model
                                       ↓
Markdown Output in timestamped directory
```

## Output Structure

```
output/
└── threat_model_20231201_143022/
    └── threat_model_Your_Document_Title.md
```

Each output file contains:
- System overview and architecture
- Trust boundaries and data flows
- Asset identification and valuation
- Threat and attack vector analysis
- Security controls and mitigations
- Risk assessment and recommendations

## Customization

### Custom Prompts

Modify the prompt files to customize the analysis approach:

**threat_model_generation.txt**: Controls the initial threat model structure
**gap_analysis.txt**: Defines how gaps are identified and prioritized

### Configuration Options

Edit `configs/default.yaml` to modify:

```yaml
llm:
  provider: openai
  model: gpt-4
  temperature: 0.7

confluence:
  timeout: 30
  max_retries: 3

threat_modeling:
  max_iterations: 10
  gap_analysis_enabled: true
```

### Azure OpenAI Configuration

For Azure OpenAI, you need to configure the following additional parameters:

```yaml
llm:
  provider: openai  # Keep as 'openai' for Azure OpenAI
  model: gpt-4
  temperature: 0.7
  base_url: https://your-resource.openai.azure.com/
  azure_deployment: your-gpt4-deployment-name
  azure_api_version: 2024-12-01-preview  # Configurable via env var
  azure_auth_method: api_key  # Options: api_key, aad, managed_identity
```

## Troubleshooting

### Common Issues

1. **Confluence Access Denied**
   - Verify your API token has appropriate permissions
   - Check that the page exists and is accessible

2. **LLM API Errors**
   - Verify your OpenAI API key is valid and has sufficient credits
   - Check rate limits and retry if necessary

3. **Image Download Failures**
   - Some Confluence images may require authentication
   - Images are downloaded but failures don't stop the process

4. **Azure Authentication Issues**
   - **AAD Authentication**: Ensure you're logged in with `az login` and have appropriate permissions
   - **Managed Identity**: Only works when running on Azure resources with Managed Identity enabled
   - **API Key Issues**: Verify your Azure OpenAI API key has the correct permissions
   - **Install azure-identity**: Run `pip install azure-identity` for AAD/Managed Identity authentication

5. **Image Handling Issues**
   - **Images not found**: The tool fetches images from Confluence page attachments via `/wiki/rest/api/content/<page_id>/child/attachment`
   - **Permission issues**: Ensure your Confluence user has permission to view page attachments
   - **Large images**: Very large images may be skipped or cause timeouts
   - **SVG Images**: SVG images are automatically converted to PNG format for LLM compatibility (requires Cairo library)
   - **SVG Conversion Setup**: See the "SVG Support" section in installation instructions for platform-specific setup
   - **Unsupported formats**: Only common image formats are downloaded (jpg, png, gif, bmp, svg, webp)
   - **Conversion failures**: If SVG conversion fails, the image will be skipped but processing continues

### Verbose Logging

Use `--verbose` flag to see detailed logs:

```bash
threat-modeler run --confluence-url "..." --verbose
```

## Security Considerations

- Store API credentials as environment variables, not in files
- Be cautious with sensitive documentation
- Review generated threat models before sharing
- Consider the data classification of your Confluence content

## Contributing

1. Fork the repository
2. Create a feature branch
3. Add tests for new functionality
4. Ensure all tests pass
5. Submit a pull request

## Deployment to GitHub

To deploy this project to your private GitHub repository:

1. **Install Git**: Download from [git-scm.com](https://git-scm.com/downloads)

2. **Create a private repository** on GitHub (don't initialize with README or .gitignore)

3. **Run the deployment script**:
   ```bash
   cd threat_modeler
   .\deploy_to_github.ps1
   ```

4. **Follow the prompts** to enter your repository details

For detailed instructions, see [DEPLOYMENT_GUIDE.md](DEPLOYMENT_GUIDE.md)

## License

MIT License - see LICENSE file for details

## Support

- **Issues**: [GitHub Issues](https://github.com/your-org/threat-modeler/issues)
- **Documentation**: [Read the Docs](https://threat-modeler.readthedocs.io/)
- **Discussions**: [GitHub Discussions](https://github.com/your-org/threat-modeler/discussions)
