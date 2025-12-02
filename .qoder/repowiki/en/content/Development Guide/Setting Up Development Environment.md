# Setting Up Development Environment

<cite>
**Referenced Files in This Document**   
- [package.json](file://package.json)
- [tsconfig.json](file://tsconfig.json)
- [tsconfig.base.json](file://tsconfig.base.json)
- [vite.config.ts](file://vite.config.ts)
- [README.md](file://README.md)
- [CONTRIBUTING.md](file://CONTRIBUTING.md)
- [chat-lib/package.json](file://chat-lib/package.json)
</cite>

## Table of Contents
1. [Introduction](#introduction)
2. [System Requirements](#system-requirements)
3. [Installing Required Tools](#installing-required-tools)
4. [Cloning the Repository](#cloning-the-repository)
5. [Installing Dependencies](#installing-dependencies)
6. [Configuration Files Overview](#configuration-files-overview)
7. [Platform-Specific Setup](#platform-specific-setup)
8. [Troubleshooting Common Issues](#troubleshooting-common-issues)
9. [Verifying the Setup](#verifying-the-setup)
10. [Conclusion](#conclusion)

## Introduction
This document provides comprehensive instructions for setting up a development environment for the vscode-copilot-chat extension. The setup process involves installing required tools, cloning the repository, configuring the development workspace, and verifying the installation. This guide covers platform-specific considerations for Windows, macOS, and Linux, along with troubleshooting tips for common setup issues.

The vscode-copilot-chat extension is a companion to GitHub Copilot that provides conversational AI assistance within Visual Studio Code. It enables developers to receive inline coding suggestions, ask questions about their code, and perform AI-powered coding sessions. The development environment setup follows standard Node.js and TypeScript practices with specific requirements for this extension.

**Section sources**
- [README.md](file://README.md#L1-L84)
- [CONTRIBUTING.md](file://CONTRIBUTING.md#L1-L439)

## System Requirements
The vscode-copilot-chat extension has specific system requirements that must be met before beginning the development setup. These requirements ensure compatibility with the extension's features and development tools.

The minimum requirements include:
- **Node.js**: Version 22.14.0 or higher
- **Python**: Version 3.10 to 3.12
- **Git Large File Storage (LFS)**: Required for running tests
- **Visual Studio Build Tools**: Required on Windows for building with node-gyp

The extension is designed to work with the latest version of Visual Studio Code, as it releases in lockstep with VS Code due to deep UI integration. This means that older versions of VS Code may not be compatible with the latest Copilot Chat extension.

**Section sources**
- [package.json](file://package.json#L25-L28)
- [CONTRIBUTING.md](file://CONTRIBUTING.md#L70-L73)

## Installing Required Tools
To set up the development environment for vscode-copilot-chat, you need to install several essential tools. This section provides step-by-step instructions for installing Node.js, TypeScript, and other VS Code development dependencies.

### Node.js Installation
Node.js is a fundamental requirement for the vscode-copilot-chat extension. The project requires Node.js version 22.14.0 or higher, as specified in the package.json file. To install Node.js:

1. Visit the official Node.js website (https://nodejs.org)
2. Download the latest LTS version that meets or exceeds version 22.14.0
3. Run the installer and follow the on-screen instructions
4. Verify the installation by opening a terminal and running:
   ```bash
   node --version
   npm --version
   ```

### TypeScript Installation
TypeScript is used extensively in the vscode-copilot-chat codebase. While it can be installed globally, it's recommended to use the version specified in the project's package.json file. The project uses TypeScript 5.8.3, as indicated in the chat-lib/package.json file.

To install TypeScript:
```bash
npm install -g typescript@5.8.3
```

### Additional Development Dependencies
Several other tools are required for development:

- **Git LFS**: Install Git Large File Storage to handle large files in the repository
- **Python**: Install Python 3.10 to 3.12 from the official Python website
- **Visual Studio Build Tools**: On Windows, install Visual Studio Build Tools 2019 or later for node-gyp compilation

**Section sources**
- [package.json](file://package.json#L25-L28)
- [CONTRIBUTING.md](file://CONTRIBUTING.md#L70-L73)
- [chat-lib/package.json](file://chat-lib/package.json#L36)

## Cloning the Repository
To begin development on the vscode-copilot-chat extension, you need to clone the repository from GitHub. This section provides instructions for cloning the repository and setting up the local workspace.

### Repository Cloning
The vscode-copilot-chat repository is hosted on GitHub at https://github.com/microsoft/vscode-copilot-chat. To clone the repository:

1. Open a terminal or command prompt
2. Navigate to your desired development directory
3. Run the following command:
   ```bash
   git clone https://github.com/microsoft/vscode-copilot-chat.git
   ```
4. Change into the cloned directory:
   ```bash
   cd vscode-copilot-chat
   ```

### Repository Structure
The repository has a well-organized structure with the following key directories:
- **src/**: Contains the main extension source code
- **test/**: Contains unit and integration tests
- **chat-lib/**: Contains the chat and inline editing SDK
- **script/**: Contains various build and setup scripts
- **assets/**: Contains static assets like images and prompts

Understanding this structure is important for navigating the codebase and making contributions.

**Section sources**
- [README.md](file://README.md#L11-L17)
- [CONTRIBUTING.md](file://CONTRIBUTING.md#L34-L35)

## Installing Dependencies
After cloning the repository, the next step is to install the required npm packages and dependencies. This section covers the installation process and any special considerations.

### npm Package Installation
To install the dependencies, navigate to the root directory of the cloned repository and run:

```bash
npm install
```

This command reads the package.json file and installs all the dependencies listed in it. The installation process may take several minutes depending on your internet connection and system performance.

### Post-Installation Scripts
The repository includes post-installation scripts that run automatically after npm install. These scripts handle additional setup tasks such as:
- Setting up environment variables
- Configuring development tools
- Preparing test environments

The chat-lib package also has a postinstall script that runs automatically:
```json
"postinstall": "tsx script/postinstall.js"
```

### Token Acquisition
After installing dependencies, you need to acquire a token for development:
```bash
npm run get_token
```

This step is necessary for accessing certain features and APIs during development.

**Section sources**
- [package.json](file://package.json#L1-L5257)
- [CONTRIBUTING.md](file://CONTRIBUTING.md#L77-L79)
- [chat-lib/package.json](file://chat-lib/package.json#L67)

## Configuration Files Overview
The vscode-copilot-chat extension uses several configuration files to manage the build process, TypeScript compilation, and development environment. Understanding these files is crucial for effective development.

### tsconfig.json
The tsconfig.json file configures the TypeScript compiler for the project. Key settings include:

- **extends**: Inherits base configuration from tsconfig.base.json
- **compilerOptions**: Specifies compilation options such as:
  - jsx: Set to "react" for JSX support
  - jsxFactory and jsxFragmentFactory: Custom factories for JSX elements
  - rootDir: Sets the root directory for input files
  - types: Specifies type definitions to include
- **include**: Specifies which files and directories to include in compilation
- **exclude**: Specifies which files and directories to exclude from compilation

The configuration is optimized for the extension's needs, including shims for VS Code API types during testing.

**Section sources**
- [tsconfig.json](file://tsconfig.json#L1-L40)
- [tsconfig.base.json](file://tsconfig.base.json#L1-L23)

### vite.config.ts
The vite.config.ts file configures the Vite build tool used for development and testing. Key aspects include:

- **Test Configuration**: Sets up Vitest for running tests with specific include and exclude patterns
- **Alias Configuration**: Creates aliases for module imports, particularly for VS Code API shims
- **Plugins**: Includes essential plugins like:
  - vite-plugin-wasm: For WebAssembly support
  - vite-plugin-top-level-await: For top-level await support
- **Environment Variables**: Loads environment variables for different build modes

The configuration ensures efficient development builds and proper test execution.

**Section sources**
- [vite.config.ts](file://vite.config.ts#L1-L40)

### package.json
The package.json file contains metadata about the project and configuration for npm. Important sections include:

- **engines**: Specifies required Node.js and npm versions
- **contributes**: Defines extension contributions like language model tools and chat participants
- **scripts**: Contains npm scripts for development tasks
- **dependencies**: Lists production dependencies
- **devDependencies**: Lists development dependencies

The file also includes configuration for various tools and services used by the extension.

**Section sources**
- [package.json](file://package.json#L1-L5257)

## Platform-Specific Setup
The development setup for vscode-copilot-chat has some platform-specific considerations for Windows, macOS, and Linux. This section covers the unique requirements and setup steps for each platform.

### Windows Setup
On Windows, additional setup steps are required:

1. Run PowerShell as administrator and execute:
   ```powershell
   Set-ExecutionPolicy Unrestricted
   ```
   This allows the execution of PowerShell scripts during the build process.

2. Install Visual Studio Build Tools 2019 or later, which are required for node-gyp to compile native modules.

3. Ensure Git is properly configured with LFS support.

The CONTRIBUTING.md file specifically mentions these Windows requirements for building with node-gyp.

### macOS and Linux Setup
For macOS and Linux, the setup is generally more straightforward:

1. Ensure you have the required Node.js version installed
2. Install Python 3.10-3.12 using your system's package manager
3. Install Git LFS using your package manager or from the official website

On Linux, you may also need to install build tools:
```bash
# Ubuntu/Debian
sudo apt-get install build-essential

# CentOS/RHEL
sudo yum groupinstall "Development Tools"
```

### Cross-Platform Considerations
The project supports development under Windows Subsystem for Linux (WSL) by following the VS Code setup instructions. This allows Windows users to leverage a Linux-like environment for development.

All platform-specific scripts (simulate.ps1 for Windows and simulate.sh for Unix-like systems) are provided to ensure consistent behavior across platforms.

**Section sources**
- [CONTRIBUTING.md](file://CONTRIBUTING.md#L76-L83)
- [script/simulate.ps1](file://script/simulate.ps1)
- [script/simulate.sh](file://script/simulate.sh)

## Troubleshooting Common Issues
This section addresses common issues that may arise during the development environment setup and provides solutions for resolving them.

### Dependency Conflicts
Dependency conflicts can occur when there are version mismatches between packages. To resolve:

1. Clear npm cache:
   ```bash
   npm cache clean --force
   ```

2. Delete node_modules and package-lock.json:
   ```bash
   rm -rf node_modules package-lock.json
   ```

3. Reinstall dependencies:
   ```bash
   npm install
   ```

Ensure you are using the exact Node.js version specified in the package.json file, as version mismatches can cause compatibility issues.

### Permission Errors
Permission errors may occur during installation or execution:

- On Unix-like systems, avoid using sudo with npm. Instead, fix npm permissions:
  ```bash
  mkdir ~/.npm-global
  npm config set prefix '~/.npm-global'
  ```
  Then add ~/.npm-global/bin to your PATH.

- On Windows, run PowerShell or Command Prompt as administrator when necessary, especially for the initial setup.

### Environment Variable Configuration
Proper environment variable configuration is essential:

1. Ensure Node.js and npm are in your system PATH
2. Verify Python is accessible from the command line
3. Set any required environment variables as specified in the documentation

For VS Code development, ensure the VS Code executable is in your PATH or properly configured in your development environment.

### Git LFS Issues
If tests fail due to missing files, ensure Git LFS is properly installed and initialized:

```bash
git lfs install
git lfs pull
```

This ensures large files required for testing are properly downloaded.

**Section sources**
- [CONTRIBUTING.md](file://CONTRIBUTING.md#L86-L87)
- [package.json](file://package.json#L27-L28)

## Verifying the Setup
After completing the setup process, it's important to verify that everything is working correctly. This section provides steps to test the installation and launch the extension in development mode.

### Build and Launch
To verify the setup, perform a test build and launch the extension:

1. Run the build task:
   ```bash
   npm run build
   ```
   Or use the VS Code task "Launch Copilot Extension - Watch Mode" (Cmd+Shift+P).

2. Start debugging the extension using the "Launch Copilot Extension - Watch Mode" launch configuration.

3. This will compile the code and launch a new VS Code window with the extension loaded.

### Running Tests
Verify the setup by running the test suite:

1. Run unit tests:
   ```bash
   npm run test:unit
   ```

2. Run extension integration tests:
   ```bash
   npm run test:extension
   ```

3. Run simulation tests (requires proper token setup):
   ```bash
   npm run simulate
   ```

Successful test execution confirms that the development environment is properly configured.

### Development Mode Features
When launched in development mode, the extension provides additional debugging features:

- **Show Chat Debug View**: Displays details of requests made by Copilot Chat
- Various debug commands accessible through the Command Palette
- Enhanced logging and diagnostic capabilities

These features help developers understand and troubleshoot the extension's behavior.

**Section sources**
- [CONTRIBUTING.md](file://CONTRIBUTING.md#L79-L82)
- [package.json](file://package.json#L1837-L2371)

## Conclusion
Setting up the development environment for the vscode-copilot-chat extension requires careful attention to the specific requirements and configuration. By following the steps outlined in this document, developers can successfully install the necessary tools, configure their workspace, and begin contributing to the extension.

The key aspects of the setup process include installing the required Node.js version, cloning the repository, installing dependencies, and understanding the configuration files. Platform-specific considerations, particularly for Windows users, must be addressed to ensure a smooth development experience.

Once the environment is set up, developers can verify their installation by building the extension, running tests, and launching it in development mode. The comprehensive tooling and configuration provided by the project make it possible to effectively develop and debug the vscode-copilot-chat extension.

With the development environment properly configured, contributors can focus on enhancing the AI-powered coding assistance features that make GitHub Copilot valuable to developers worldwide.