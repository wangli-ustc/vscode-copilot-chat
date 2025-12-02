# Proposed APIs

<cite>
**Referenced Files in This Document**   
- [package.json](file://package.json)
- [extHost.api.impl.ts](file://src/extension/prompts/node/test/fixtures/extHost.api.impl.ts)
- [vscode.proposed.extensionsAny.d.ts](file://src/extension/vscode.proposed.extensionsAny.d.ts)
</cite>

## Table of Contents
1. [Introduction](#introduction)
2. [Purpose of extensionsAny API](#purpose-of-extensionsany-api)
3. [Core API Additions](#core-api-additions)
4. [Use Cases](#use-cases)
5. [Experimental Nature and Stability](#experimental-nature-and-stability)
6. [Feature Detection and Fallback Patterns](#feature-detection-and-fallback-patterns)
7. [Conclusion](#conclusion)

## Introduction
This document details the proposed `extensionsAny` namespace APIs extended by the VS Code Copilot extension. These APIs enable cross-extension host communication and discovery, allowing extensions to access functionality across different extension host environments such as webviews and notebook extension hosts. The document focuses on the `getExtension` overload with the `includeDifferentExtensionHosts` parameter and the `allAcrossExtensionHosts` property, explaining their purpose, use cases, experimental nature, and implementation patterns.

**Section sources**
- [package.json](file://package.json#L91)
- [extHost.api.impl.ts](file://src/extension/prompts/node/test/fixtures/extHost.api.impl.ts#L425-L457)

## Purpose of extensionsAny API
The `extensionsAny` API proposal addresses the need for extensions to discover and interact with other extensions across different extension host environments within VS Code. In modern VS Code architecture, extensions can run in various contexts including the main extension host, webview extension hosts, and notebook extension hosts. The `extensionsAny` APIs provide a mechanism for extensions to:

- Discover extensions running in different extension host environments
- Access functionality from extensions that are not in the same extension host context
- Enable cross-context communication between extensions
- Support scenarios where functionality needs to be shared between different parts of the VS Code ecosystem

This capability is particularly important for AI-powered features like Copilot that need to integrate with various extension contexts and provide consistent functionality across different editor environments.

**Section sources**
- [extHost.api.impl.ts](file://src/extension/prompts/node/test/fixtures/extHost.api.impl.ts#L425-L457)

## Core API Additions
The `extensionsAny` proposal introduces two key additions to the VS Code extension API:

### getExtension with includeDifferentExtensionHosts parameter
The `getExtension` method has been extended with an optional `includeFromDifferentExtensionHosts` boolean parameter:

```typescript
getExtension(extensionId: string, includeFromDifferentExtensionHosts?: boolean): vscode.Extension<any> | undefined
```

When `includeFromDifferentExtensionHosts` is set to `true`, the method can return extensions that are running in different extension host environments, not just the current host. If the `extensionsAny` proposed API is not enabled, this parameter is ignored and the method behaves as the standard `getExtension` call.

### allAcrossExtensionHosts property
The `allAcrossExtensionHosts` property provides access to all extensions across all extension hosts:

```typescript
get allAcrossExtensionHosts(): vscode.Extension<any>[]
```

This property returns an array of all extensions, with extensions from different extension hosts marked appropriately. Access to this property requires the `extensionsAny` proposed API to be enabled, as indicated by the `checkProposedApiEnabled` call in the implementation.

Additionally, the `onDidChange` event has been enhanced to include changes from all extension hosts when the `extensionsAny` API is enabled, providing a unified event stream for extension changes across all contexts.

**Section sources**
- [extHost.api.impl.ts](file://src/extension/prompts/node/test/fixtures/extHost.api.impl.ts#L425-L463)
- [vscode.proposed.extensionsAny.d.ts](file://src/extension/vscode.proposed.extensionsAny.d.ts)

## Use Cases
The `extensionsAny` APIs support several important use cases for extension integration:

### Accessing Copilot Functionality from Webviews
Extensions running in webview contexts can discover and interact with the Copilot extension, enabling AI-powered features within custom webview-based UIs. This allows webview extensions to leverage Copilot's capabilities for code suggestions, explanations, and other AI-assisted development features.

### Notebook Extension Integration
Notebook extensions can access functionality from extensions running in different contexts, enabling rich integration between notebook environments and other VS Code extensions. This is particularly valuable for data science and educational scenarios where notebooks need to interact with various tools and services.

### Cross-Host Extension Discovery
Extensions can discover and communicate with other extensions regardless of their host environment, enabling more sophisticated extension ecosystems where functionality can be shared and composed across different contexts.

### Unified Extension Management
The ability to access all extensions across all hosts enables scenarios where an extension needs to provide a unified view or management interface for multiple extensions running in different contexts.

**Section sources**
- [extHost.api.impl.ts](file://src/extension/prompts/node/test/fixtures/extHost.api.impl.ts#L425-L457)

## Experimental Nature and Stability
The `extensionsAny` APIs are currently proposed and experimental, which means:

- **No stability guarantees**: These APIs may change or be removed in future VS Code releases
- **Breaking changes possible**: Consumers should expect potential breaking changes between versions
- **Feature flags required**: The APIs must be explicitly enabled through the `enabledApiProposals` field in package.json
- **Limited availability**: The APIs are only available when explicitly requested by extensions

The experimental nature is evident in the implementation, where API availability is checked using `isProposedApiEnabled` and `checkProposedApiEnabled` functions. These checks ensure that the extended functionality is only available when the extension has declared its dependency on the proposed API.

```json
"enabledApiProposals": [
    "extensionsAny",
    "newSymbolNamesProvider",
    "interactive",
    // ... other proposed APIs
]
```

Extensions using these APIs should be prepared for potential changes and should implement appropriate fallback mechanisms.

**Section sources**
- [package.json](file://package.json#L91-L140)
- [extHost.api.impl.ts](file://src/extension/prompts/node/test/fixtures/extHost.api.impl.ts#L426-L427)

## Feature Detection and Fallback Patterns
To safely use the proposed `extensionsAny` APIs, client extensions should implement feature detection and provide appropriate fallback behavior:

### Feature Detection
Extensions should check for API availability before using the extended functionality:

```typescript
// The implementation automatically handles feature detection
// When extensionsAny is not enabled, includeFromDifferentExtensionHosts is ignored
const extension = vscode.extensions.getExtension('publisher.extension', true);
```

### Fallback Implementation
When the proposed API is not available, extensions should provide alternative functionality or gracefully degrade:

```typescript
// Pattern 1: Check if the returned extension is from a different host
const extension = vscode.extensions.getExtension('publisher.extension', true);
if (extension) {
    if (extension.isFromDifferentExtensionHost) {
        // Handle cross-host extension
        await extension.activate();
        // Use extension API
    } else {
        // Handle same-host extension
        await extension.activate();
        // Use extension API
    }
} else {
    // Extension not found - provide alternative functionality
    // or show appropriate message to user
}
```

### Defensive Programming
Extensions should be designed to function without the proposed APIs:

```typescript
// Always have a fallback for when cross-host discovery is not available
const allExtensions = vscode.extensions.allAcrossExtensionHosts || vscode.extensions.all;
for (const ext of allExtensions) {
    // Process extensions
}
```

### Progressive Enhancement
Implement functionality that works with standard APIs and enhances the experience when proposed APIs are available:

```typescript
// Base functionality using standard APIs
function getRelevantExtensions() {
    const localExtensions = vscode.extensions.all;
    // Return local extensions
    return localExtensions;
}

// Enhanced functionality when proposed API is available
async function getEnhancedExtensions() {
    if (vscode.extensions.allAcrossExtensionHosts) {
        // Include extensions from all hosts
        return vscode.extensions.allAcrossExtensionHosts;
    }
    return getRelevantExtensions();
}
```

**Section sources**
- [extHost.api.impl.ts](file://src/extension/prompts/node/test/fixtures/extHost.api.impl.ts#L426-L428)
- [package.json](file://package.json#L91)

## Conclusion
The `extensionsAny` proposed APIs represent an important evolution in VS Code's extension model, enabling richer integration between extensions running in different contexts. By providing mechanisms for cross-extension host discovery and communication, these APIs open up new possibilities for extension integration, particularly for AI-powered features like Copilot that need to operate across various editor environments.

While currently experimental, these APIs address real needs in the VS Code ecosystem and provide a foundation for more sophisticated extension interactions. Extensions that use these APIs should implement proper feature detection and fallback mechanisms to ensure compatibility across different VS Code versions and configurations.

As these APIs mature, they have the potential to significantly enhance the extensibility of VS Code, enabling new scenarios for cross-context functionality and more seamless integration between different parts of the editor ecosystem.