# Behavior-Driven Testing Plugin

Guides writing of behavior-driven Go unit tests that prioritize testing meaningful behaviors over chasing code coverage.

## What It Does

This plugin provides a skill that automatically triggers when writing, reviewing, or planning Go unit tests. It enforces:

- **Gherkin-style test naming**: `"When <precondition>, it should <expected behavior>"`
- **Gomega assertions**: Expressive matchers that clearly communicate intent
- **Table-driven tests**: Idiomatic Go test structure
- **Behavior-first thinking**: Test what callers observe, not implementation details
- **Envtest guidance**: For Kubernetes API/CRD validation testing

## Core Philosophy

Before writing any test case, ask: **"What behavior would break if this code were wrong?"**

Coverage is a compass, not a target. A test suite full of shallow assertions that happens to touch every line is worse than a smaller suite that deeply validates the behaviors users and callers actually depend on.

## Installation

### From Claude Code Plugin Marketplace

```bash
/plugin marketplace add bryan-cox/personal-claude-skills
/plugin install behavior-driven-testing@personal-claude-skills
```

### Manual Installation

Clone the repository and link it to your Claude Code configuration:

```bash
git clone https://github.com/bryan-cox/personal-claude-skills.git
```

## Triggers

The skill activates automatically when working with:

- `**/*_test.go` - Go test files
- `**/tests/**/*.yaml` - Test YAML files (envtests)
- `test/envtest/**` - Envtest suites

## Example

```go
func TestReconcileWidget(t *testing.T) {
    tests := []struct {
        name     string
        widget   Widget
        wantErr  bool
        wantType WidgetType
    }{
        {
            name:     "When widget has valid config, it should reconcile successfully",
            widget:   validWidget(),
            wantType: WidgetTypeActive,
        },
        {
            name:    "When widget config is missing required field, it should return a validation error",
            widget:  widgetMissingName(),
            wantErr: true,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            g := NewWithT(t)

            result, err := ReconcileWidget(tt.widget, defaultOptions())
            if tt.wantErr {
                g.Expect(err).To(HaveOccurred())
                return
            }
            g.Expect(err).ToNot(HaveOccurred())
            g.Expect(result.Type).To(Equal(tt.wantType))
        })
    }
}
```
