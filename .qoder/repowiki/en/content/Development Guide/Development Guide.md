# Development Guide

<cite>
**Referenced Files in This Document**   
- [README.md](file://README.md)
- [CONTRIBUTING.md](file://CONTRIBUTING.md)
- [package.json](file://package.json)
- [tsconfig.json](file://tsconfig.json)
- [eslint.config.mjs](file://eslint.config.mjs)
- [vite.config.ts](file://vite.config.ts)
- [chat-lib/vitest.config.ts](file://chat-lib/vitest.config.ts)
- [lint-staged.config.js](file://lint-staged.config.js)
- [script/simulate.sh](file://script/simulate.sh)
- [script/simulate.ps1](file://script/simulate.ps1)
</cite>

## Table of Contents
1. [Introduction](#introduction)
2. [Development Environment Setup](#development-environment-setup)
3. [Build and Testing Processes](#build-and-testing-processes)
4. [Debugging Guidance](#debugging-guidance)
5. [Contribution Workflow](#contribution-workflow)
6. [Common Development Tasks](#common-development-tasks)
7. [Best Practices](#best-practices)

## Introduction
This development guide provides comprehensive instructions for contributing to the vscode-copilot-chat extension. The guide covers setting up the development environment, building and testing the extension, debugging techniques, contribution workflows, and best practices for code organization and documentation. The vscode-copilot-chat extension is a companion to GitHub Copilot that provides conversational AI assistance in Visual Studio Code, enabling features like AI-powered coding sessions, inline suggestions, and chat-based code assistance.

**Section sources**
- [README.md](file://README.md#L1-L84)

## Development Environment Setup

### Required Tools and Dependencies
To develop the vscode-copilot-chat extension, you need the following tools and dependencies:

- **Node.js 22.x**: The extension requires Node.js version 22.x for development and testing.
- **Python 3.10-3.12**: Required for running tests and certain development scripts.
- **Git Large File Storage (LFS)**: Necessary for running tests that depend on large files.
- **Visual Studio Build Tools 2019+** (Windows): Required for building with node-gyp on Windows systems.
- **PowerShell** (Windows): Required for running setup scripts with appropriate execution policies.

### First-time Setup
Follow these steps to set up your development environment:

1. On Windows, run `Set-ExecutionPolicy Unrestricted` as administrator in PowerShell to allow script execution.
2. Install dependencies using `npm install`.
3. Obtain an authentication token by running `npm run get_token`.
4. Start the development environment using the "Launch Copilot Extension - Watch Mode" task (Ctrl+Shift+B) or debug configuration.

For development under Windows Subsystem for Linux (WSL), follow the VS Code setup instructions for self-hosting on WSL.

**Section sources**
- [CONTRIBUTING.md](file://CONTRIBUTING.md#L69-L83)

## Build and Testing Processes

### Build Process
The extension uses Vite for building and bundling. The build configuration is defined in `vite.config.ts`, which sets up aliases, plugins, and test configurations. The build process automatically handles TypeScript compilation and module resolution.

### Testing Framework
The extension employs multiple testing strategies:

- **Unit Tests**: Run in Node.js using Vitest, covering isolated components and utilities.
- **Integration Tests**: Execute within VS Code itself, testing extension functionality in the actual editor environment.
- **Simulation Tests**: Comprehensive tests that interact with Copilot API endpoints and invoke LLMs, requiring expensive computations.

### Running Tests
Execute the different test types using these npm scripts:

```bash
# Run unit tests
npm run test:unit

# Run integration tests in VS Code
npm run test:extension

# Run simulation tests (requires authentication token)
npm run simulate
```

Simulation tests are particularly important as they validate the extension's interaction with the Copilot service. These tests are stochastic due to LLM randomness, so results are snapshotted in `test/simulation/baseline.json`. The tests use caching in `test/simulation/cache` to ensure deterministic results on subsequent runs.

To ensure tests pass in CI, verify the cache is populated:
```bash
npm run simulate-require-cache
```

If test results change and you want to update the baseline:
```bash
npm run simulate-update-baseline
```

**Section sources**
- [CONTRIBUTING.md](file://CONTRIBUTING.md#L85-L122)
- [vite.config.ts](file://vite.config.ts#L1-L40)
- [chat-lib/vitest.config.ts](file://chat-lib/vitest.config.ts#L1-L21)

## Debugging Guidance

### Frontend UI Issues
For debugging frontend UI issues in the extension:

1. Use the "Show Chat Debug View" command to inspect requests made by Copilot Chat.
2. The debug view displays a tree with entries for each request, showing the prompt sent to the model, enabled tools, response, and other details.
3. Export request logs using right-click > "Export As..." for analysis and troubleshooting.
4. Review the prompt rendering to ensure it matches expectations, as this is critical for proper AI behavior.

### Backend Service Problems
When debugging backend service issues:

1. Check the Developer Tools Console (Help > Toggle Developer Tools) for errors.
2. Verify network requests and responses in the Network tab.
3. Use the simulation test framework to reproduce issues in a controlled environment.
4. Examine the request logger output to understand the flow of data between the extension and Copilot services.

**Note**: Request logs may contain personal information such as file contents or terminal output. Review carefully before sharing with others.

**Section sources**
- [CONTRIBUTING.md](file://CONTRIBUTING.md#L300-L308)

## Contribution Workflow

### Code Review Process
The contribution workflow follows standard GitHub practices with specific requirements for this extension:

1. Create issues following the guidelines in CONTRIBUTING.md.
2. Search for existing issues before creating new ones.
3. Provide detailed information including VS Code and extension versions, operating system, LLM model, reproducible steps, expected vs. actual behavior, and relevant code snippets.
4. Use the built-in "Report Issue" tool in VS Code to automatically include system information.

### Quality Standards
Contributions must adhere to the following quality standards:

- Follow the project's TypeScript/JavaScript guidelines and React/JSX conventions.
- Maintain proper layer separation as defined in the code structure.
- Use the `base/common` utilities from the VS Code repository when appropriate.
- Comply with eslint rules defined in `eslint.config.mjs`.
- Format code using `tsfmt` as configured in `tsfmt.json`.

The project uses lint-staged to automatically format and lint files before commits:
```javascript
// lint-staged.config.js
module.exports = {
  '!({.esbuild.ts,test/simulation/fixtures/**,test/scenarios/**,.vscode/extensions/**,**/vscode.proposed.*})*{.ts,.js,.tsx}': async (files) => {
    const filesToLint = await removeIgnoredFiles(files);
    return [
      `npm run tsfmt -- ${filesToLint}`,
      `eslint --max-warnings=0 ${filesToLint}`
    ];
  },
};
```

**Section sources**
- [CONTRIBUTING.md](file://CONTRIBUTING.md#L30-L66)
- [eslint.config.mjs](file://eslint.config.mjs#L1-L529)
- [lint-staged.config.js](file://lint-staged.config.js#L1-L28)

## Common Development Tasks

### Adding New Features
To add new features to the extension:

1. Identify the appropriate layer (common, vscode, node, vscode-node, worker, or vscode-worker) based on the runtime requirements.
2. Implement the feature following the project's architecture patterns.
3. Register contributions and services in the appropriate files:
   - `./extension/extension/vscode/contributions.ts` for cross-runtime features
   - `./extension/extension/vscode-node/contributions.ts` for node.js-specific features
   - `./extension/extension/vscode-worker/contributions.ts` for web worker-specific features
4. Write comprehensive tests covering unit, integration, and simulation scenarios.
5. Update documentation and ensure compliance with coding standards.

### Fixing Bugs
When fixing bugs:

1. Reproduce the issue using the provided steps.
2. Use the debug tools to identify the root cause.
3. Implement the fix while maintaining backward compatibility.
4. Add or update tests to prevent regression.
5. Follow the API update guidelines when making changes that affect compatibility.

### Improving Performance
For performance improvements:

1. Analyze the current implementation using profiling tools.
2. Optimize critical paths, particularly in the prompt rendering and API communication.
3. Leverage caching mechanisms where appropriate.
4. Minimize unnecessary computations and memory allocations.
5. Test performance changes using the simulation test framework.

**Section sources**
- [CONTRIBUTING.md](file://CONTRIBUTING.md#L200-L267)

## Best Practices

### Code Organization
The extension follows a layered architecture with clear separation of concerns:

- **common**: JavaScript and built-in APIs only, with access to VS Code API types but no runtime access.
- **vscode**: Runtime access to VS Code APIs.
- **node**: Node.js APIs and modules.
- **vscode-node**: VS Code and Node.js APIs.
- **worker**: Web Worker APIs.
- **vscode-worker**: VS Code and Web Worker APIs.

Top-level folders organize code by functionality:
- **src/util**: Utility code usable across the codebase.
- **src/platform**: Services for implementing extension features (telemetry, configuration, search).
- **src/extension**: Main implementation of extension functionality.

### Testing
Adhere to these testing best practices:

- Write unit tests for isolated components.
- Create integration tests for features that interact with VS Code APIs.
- Develop simulation tests for end-to-end validation of AI-powered features.
- Ensure test coverage for edge cases and error conditions.
- Update baselines when legitimate changes occur in simulation test results.

### Documentation
Follow these documentation practices:

- Use the TSX-based framework for prompt composition.
- Document new features and APIs comprehensively.
- Update README and CONTRIBUTING documentation when making significant changes.
- Provide clear examples and usage instructions.
- Maintain consistency in code comments and documentation style.

**Section sources**
- [CONTRIBUTING.md](file://CONTRIBUTING.md#L200-L229)
- [package.json](file://package.json#L1-L5257)