# Contributing to ClientAI

Thank you for your interest in contributing to ClientAI! This guide is meant to make it easy for you to get started.

## Setting Up Your Development Environment

### Cloning the Repository

Start by forking and cloning the ClientAI repository:

```sh
git clone https://github.com/YOUR-GITHUB-USERNAME/clientai.git

cd clientai
```

### Using UV for Dependency Management

ClientAI uses UV for managing dependencies. If you don't have UV installed, follow the instructions on the [official UV website](https://docs.astral.sh/uv/getting-started/installation/).

Once UV is installed, navigate to the cloned repository and install the dependencies:

```sh
uv sync
```

UV creates a virtual environment by itself (.venv) so you do not need to do it by hand or using another tool. Additionally, remember calling every command from UV (`uv run` in front of your command), so the venv will be considered. More examples below.

## Making Contributions

### Coding Standards

- Follow PEP 8 guidelines.
- Write meaningful tests for new features or bug fixes.

### Testing with Pytest

ClientAI uses pytest for testing. Run tests using:

```sh
uv run pytest
```

### Linting

Use mypy for type checking:

```sh
uv run mypy clientai
```

Use ruff for style:

```sh
uv run ruff check --fix
uv run ruff format
```

Ensure your code passes linting before submitting.

## Submitting Your Contributions

### Creating a Pull Request

After making your changes:

- Push your changes to your fork.
- Open a pull request with a clear description of your changes.
- Update the README.md if necessary.

### Code Reviews

- Address any feedback from code reviews.
- Once approved, your contributions will be merged into the main branch.

## Code of Conduct

Please adhere to our [Code of Conduct](CODE_OF_CONDUCT.md) to maintain a welcoming and inclusive environment.

Thank you for contributing to ClientAI🚀
