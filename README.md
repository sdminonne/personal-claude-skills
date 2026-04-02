# personal-claude-skills

A place for me to share personal Claude Code skills I use on a daily basis that might help others as well but I don't want to merge them into a formal repo just yet.

## Installation

1. Open Claude Code and run `/plugin`
2. Select **Add marketplace**
3. Enter the repository URL: `https://github.com/bryan-cox/personal-claude-skills`
4. Select **Install plugin**
5. Choose the plugin you want to install

## Available Plugins

### behavior-driven-testing

Guides writing of behavior-driven Go unit tests that prioritize testing meaningful behaviors over chasing code coverage.

**What it does:**
- Enforces Gherkin-style test naming: `"When <precondition>, it should <expected behavior>"`
- Promotes gomega assertions with expressive matchers
- Guides table-driven test structure (idiomatic Go)
- Covers fake clients, mocking patterns, and envtest guidance for Kubernetes API testing
- Encourages testing behaviors, not implementation details
