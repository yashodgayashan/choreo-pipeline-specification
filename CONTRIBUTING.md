# Contributing to Choreo Pipeline Specification

Thank you for your interest in contributing to the Choreo Pipeline Specification! This document provides guidelines and instructions for contributing.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [How to Contribute](#how-to-contribute)
- [Development Setup](#development-setup)
- [Contribution Process](#contribution-process)
- [Style Guidelines](#style-guidelines)
- [Testing](#testing)
- [Documentation](#documentation)

## Code of Conduct

We are committed to providing a welcoming and inclusive environment. Please:

- Be respectful and considerate
- Welcome newcomers and help them get started
- Focus on what is best for the community
- Show empathy towards other community members

## How to Contribute

### Reporting Issues

Before creating an issue:
1. Check existing issues to avoid duplicates
2. Use issue templates when available
3. Provide clear, detailed information

Include in your issue:
- Pipeline specification version
- Example YAML that demonstrates the issue
- Expected behavior
- Actual behavior
- Steps to reproduce

### Suggesting Enhancements

Enhancement suggestions are welcome! Please:
1. Check if the enhancement has already been suggested
2. Explain the use case and benefits
3. Provide example configurations
4. Consider backward compatibility

### Contributing Code

We accept contributions for:
- Bug fixes
- New examples
- Documentation improvements
- New features (with prior discussion)
- Test improvements

## Development Setup

### Prerequisites

- Git
- Python 3.8+
- Node.js 16+ (for validation tools)
- MkDocs (for documentation)

### Setup Steps

1. Fork the repository:
```bash
git clone https://github.com/wso2/choreo-pipeline-specification.git
cd choreo-pipeline-specification
```

2. Create a virtual environment:
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. Install dependencies:
```bash
pip install -r requirements.txt
npm install
```

4. Install pre-commit hooks:
```bash
pre-commit install
```

## Contribution Process

### 1. Create a Branch

```bash
git checkout -b feature/your-feature-name
# or
git checkout -b fix/issue-description
```

Branch naming conventions:
- `feature/` - New features
- `fix/` - Bug fixes
- `docs/` - Documentation only
- `example/` - New examples
- `refactor/` - Code refactoring

### 2. Make Changes

Follow these guidelines:
- Keep changes focused and atomic
- Update relevant documentation
- Add/update tests as needed
- Follow style guidelines

### 3. Validate Changes

Run validation:
```bash
# Validate YAML syntax
npm run validate

# Check JSON schema
npm run schema-check

# Run linters
npm run lint

# Test documentation build
mkdocs build
```

### 4. Commit Changes

Write clear commit messages:
```bash
git commit -m "type: brief description

Detailed explanation of what changed and why.

Fixes #123"
```

Commit types:
- `feat:` - New feature
- `fix:` - Bug fix
- `docs:` - Documentation only
- `style:` - Code style changes
- `refactor:` - Code refactoring
- `test:` - Test additions/changes
- `chore:` - Maintenance tasks

### 5. Submit Pull Request

1. Push your branch:
```bash
git push origin feature/your-feature-name
```

2. Create a pull request with:
   - Clear title and description
   - Reference to related issues
   - Examples of usage
   - Screenshots (if applicable)

3. Ensure all checks pass

4. Respond to review feedback

## Style Guidelines

### YAML Files

- Use 2 spaces for indentation
- Add comments for complex configurations
- Keep line length under 120 characters
- Use meaningful names for steps and templates

Example:
```yaml
steps:
  # Build and test the application
  - name: build-and-test
    inlineScript: |
      #!/bin/bash
      npm install
      npm test
```

### Documentation

- Use clear, concise language
- Include code examples
- Follow Markdown best practices
- Update table of contents when adding sections
- Use proper heading hierarchy

### Examples

When adding examples:
1. Place in appropriate category folder
2. Include comprehensive comments
3. Test the example thoroughly
4. Update examples/README.md
5. Keep examples realistic and practical

## Testing

### Testing Examples

Validate example files:
```bash
# Validate single file
python scripts/validate.py examples/basic/hello-world.yaml

# Validate all examples
python scripts/validate.py examples/
```

### Schema Validation

Test against JSON schema:
```bash
npm run validate-schema examples/basic/hello-world.yaml
```

### Documentation Testing

Test documentation:
```bash
# Build docs locally
mkdocs serve

# Check for broken links
linkchecker http://localhost:8000
```

## Documentation

### Adding New Documentation

1. Create markdown file in appropriate directory
2. Update navigation in mkdocs.yml
3. Include examples and use cases
4. Add to table of contents
5. Cross-reference related documents

### API Documentation

When documenting APIs:
- Include all parameters
- Specify required vs optional
- Provide examples
- Document default values
- Explain validation rules

## Review Process

### Review Criteria

Pull requests are reviewed for:
- Code quality and style compliance
- Documentation completeness
- Test coverage
- Backward compatibility
- Security considerations

### Review Timeline

- Initial review: 2-3 business days
- Follow-up reviews: 1-2 business days
- Urgent fixes: Expedited review

## Release Process

### Version Numbering

We follow Semantic Versioning (MAJOR.MINOR.PATCH):
- MAJOR: Breaking changes
- MINOR: New features (backward compatible)
- PATCH: Bug fixes

### Release Checklist

- [ ] Update version in specification
- [ ] Update CHANGELOG.md
- [ ] Update documentation
- [ ] Tag release
- [ ] Create GitHub release
- [ ] Notify users of changes

## Getting Help

If you need help:
1. Check documentation
2. Search existing issues
3. Ask in discussions
4. Contact maintainers

## Recognition

Contributors are recognized in:
- CONTRIBUTORS.md file
- Release notes
- Project documentation

## License

By contributing, you agree that your contributions will be licensed under the project's license.

## Questions?

Feel free to:
- Open a discussion
- Contact maintainers
- Join community channels

Thank you for contributing to Choreo Pipeline Specification!