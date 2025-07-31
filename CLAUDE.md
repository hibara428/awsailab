# AWS AI Lab - Claude Code Configuration

## Project Overview
This repository is designed for AWS AI research and development, focusing on leveraging Claude Code to interact with AWS services efficiently and conduct AI-related experiments.

## Commands & Scripts
- Install dependencies: `uv sync`
- Install dev tools: `uv sync --extra dev`
- Add new package: `uv add package_name`
- Test command: `uv run pytest tests/` (when tests are available)
- Lint command: `uv run ruff check .`
- Format command: `uv run ruff format .`
- Type check: `uv run mypy src/`
- Run scripts: `uv run python scripts/script_name.py`

## AWS Configuration
- AWS CLI should be configured with appropriate credentials
- Use AWS profiles when working with multiple accounts
- Default region: us-east-1 (can be overridden)

## Directory Structure
Directories will be created as needed for specific projects and experiments.

## Research Focus Areas
- AWS Bedrock integration and experimentation
- AI model deployment on AWS SageMaker
- AWS Lambda functions for AI workflows
- Cost optimization for AI workloads
- Security best practices for AI services

## MCP Servers Configuration
This repository is configured with the following MCP servers (see .mcp.json):
- **aws-docs**: AWS documentation and API references
- **aws-serverless**: Serverless development guidance (Lambda, API Gateway, etc.)
- **aws-bedrock**: Amazon Bedrock AI/ML services
- **aws-cdk**: Infrastructure as Code with CDK
- **aws-iam**: IAM policies and security configurations
- **aws-pricing**: Cost estimation and pricing analysis
- **aws-diagram**: AWS architecture diagram generation

## Claude Code Instructions
When working with this repository:
1. Always check AWS credentials and region configuration before running AWS commands
2. Use defensive programming practices - validate inputs and handle errors gracefully
3. Follow AWS security best practices - never hardcode credentials
4. Consider cost implications when creating AWS resources
5. Document any new AWS resources created for easy cleanup
6. Use appropriate AWS SDK methods rather than CLI when possible in Python scripts
7. Leverage MCP servers for up-to-date AWS guidance and best practices

## Environment Variables
- `AWS_PROFILE`: AWS profile to use
- `AWS_REGION`: AWS region for operations
- `OPENAI_API_KEY`: For OpenAI integration (if needed)
- `ANTHROPIC_API_KEY`: For Anthropic API access (if needed)

## Notes
- This is a research environment - prefer experimentation and learning
- Always clean up resources after experiments to avoid costs
- Use version control for configuration changes
- Document findings in markdown files in respective directories