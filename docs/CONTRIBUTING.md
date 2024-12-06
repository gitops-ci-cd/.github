# Contributing

Thank you for your interest in contributing! We welcome all contributions and appreciate your help in making this project better.

## Code of Conduct

Please review our [Code of Conduct](?tab=coc-ov-file) before contributing. By participating in this project, you agree to abide by its terms.

## How to Contribute

There are several ways to help us improve. As always, please [search for existing issues](/issues) before opening a new one to avoid duplicates.

### Reporting Issues

Submit a [bug report](/issues/new/choose).

### Suggesting Features

Submit a [feature request](/issues/new/choose).

### Submitting Pull Requests

1. **Fork the repository** and create your branch from `main`.
2. Follow our [Coding Standards](#coding-standards).
3. **Test** your changes to make sure they work as expected.
4. Submit a **clear and detailed PR description**, including a reference to any related issue.
5. One of our maintainers will review and provide feedback on your PR.

## Setting Up Your Environment

- Install [Brew](https://brew.sh/)
- Install [Docker Desktop](https://docs.docker.com/desktop/)
- Clone the repository and `cd` into the project directory.
- Run the following commands to set up your environment:

    ```sh
    brew bundle # refer to Brewfile for dependencies

    docker compose run --rm install
    docker compose watch
    ```

## Coding Standards

- Follow any style guidelines set by this project (e.g., Prettier, ESLint) by using `docker compose run --rm lint`.
- Keep code readable and well-documented.
- Use consistent naming conventions and clean architecture.

## Testing

Before submitting a pull request, make sure all tests pass, locally, though CI should catch failures.

- Run tests using `docker compose run --rm test`.
- Add new tests if you introduce a new feature or fix a bug.

## Additional Notes

- Please review our [Security Policy](/security/policy).
- Start or join a [discussion](/discussions) if you have questions.
- Thanks again for your contributions!
