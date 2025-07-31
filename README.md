# AWS AI Lab

A research and development environment for AWS AI services, optimized for experimentation with Claude Code integration.

## Overview

This repository provides a comprehensive setup for AWS AI research and development, featuring:

- **AWS Bedrock Integration**: Direct access to Claude and other foundation models
- **Serverless AI Workflows**: Lambda-based processing pipelines
- **Cost-Optimized Experiments**: Tools and practices for efficient resource usage
- **Security Best Practices**: IAM configurations and credential management
- **Architecture Visualization**: Automated diagram generation for AWS architectures

## Quick Start

### Prerequisites
- Python 3.8+
- AWS CLI configured with appropriate credentials
- [uv](https://docs.astral.sh/uv/) for dependency management

### Installation

```bash
# Clone the repository
git clone https://github.com/hibara428/awsailab.git
cd awsailab

# Install dependencies
uv sync

# Install development tools
uv sync --extra dev
```

### Configuration

1. **AWS Setup**: Ensure your AWS CLI is configured with appropriate credentials
2. **Environment Variables**: Set required environment variables in your shell
3. **MCP Servers**: The project includes pre-configured MCP servers for AWS services

## Project Structure

```
awsailab/
├── src/                    # Source code for experiments
├── scripts/               # Utility scripts
├── generated-diagrams/    # Auto-generated architecture diagrams
├── tests/                 # Test suite
├── CLAUDE.md             # Claude Code configuration
└── pyproject.toml        # Project dependencies
```

## Available Commands

```bash
# Development
uv run pytest tests/           # Run tests
uv run ruff check .           # Lint code
uv run ruff format .          # Format code
uv run mypy src/              # Type checking

# Scripts
uv run python scripts/script_name.py

# Dependencies
uv add package_name           # Add new package
```

## MCP Servers

This project leverages MCP servers for enhanced AWS development:

- **aws-diagram**: Architecture diagram generation using Python diagrams
- **aws-knowledge**: AWS documentation, API references, and search capabilities

## Sample Architectures

The `generated-diagrams/` directory contains example AWS architectures:

- **S3-Bedrock Image Processing**: Serverless image analysis pipeline using Claude models

## Security & Best Practices

- Never commit AWS credentials to version control
- Use AWS profiles for multi-account access
- Follow least-privilege IAM principles
- Clean up resources after experiments to avoid costs
- Use defensive programming practices

## Contributing

1. Follow the existing code style and conventions
2. Add tests for new functionality
3. Update documentation as needed
4. Use meaningful commit messages

## Research Focus Areas

- AWS Bedrock model integration and optimization
- Serverless AI workflow architectures
- Cost optimization strategies for AI workloads
- Security patterns for AI applications
- Multi-modal AI processing pipelines

## License

This project is for research and educational purposes.

---

For detailed configuration and usage instructions, see [CLAUDE.md](CLAUDE.md).
