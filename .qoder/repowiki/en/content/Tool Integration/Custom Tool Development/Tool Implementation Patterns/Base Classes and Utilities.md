# Base Classes and Utilities

<cite>
**Referenced Files in This Document**   
- [abstractReplaceStringTool.tsx](file://src/extension/tools/node/abstractReplaceStringTool.tsx)
- [toolsService.ts](file://src/extension/tools/common/toolsService.ts)
- [toolUtils.ts](file://src/extension/tools/common/toolUtils.ts)
- [editFileToolUtils.tsx](file://src/extension/tools/node/editFileToolUtils.tsx)
- [toolsRegistry.ts](file://src/extension/tools/common/toolsRegistry.ts)
- [replaceStringTool.tsx](file://src/extension/tools/node/replaceStringTool.tsx)
- [multiReplaceStringTool.tsx](file://src/extension/tools/node/multiReplaceStringTool.tsx)
- [toolsService.ts](file://src/extension/tools/vscode-node/toolsService.ts)
</cite>

## Table of Contents
1. [Introduction](#introduction)
2. [AbstractReplaceStringTool Base Class](#abstractreplacestringtool-base-class)
3. [ToolsService Registry and Execution Manager](#toolsservice-registry-and-execution-manager)
4. [Tool Utilities](#tool-utilities)
5. [Edit File Tool Utilities](#edit-file-tool-utilities)
6. [Concrete Tool Implementations](#concrete-tool-implementations)
7. [Best Practices for Extension and Usage](#best-practices-for-extension-and-usage)
8. [Conclusion](#conclusion)

## Introduction
This document provides a comprehensive analysis of the base classes and utilities used in tool implementation within the vscode-copilot-chat extension. The architecture is designed around a modular system where tools extend base classes and leverage shared utilities to maintain consistency and reduce code duplication. The core components include the AbstractReplaceStringTool as a foundational base class for text modification tools, the toolsService as the central registry and execution manager, and various utility modules that provide common operations for string manipulation, path handling, and error formatting. This documentation will thoroughly explain these components, their interfaces, lifecycle methods, and extension points, along with concrete examples from the codebase showing how these base components are extended and used in actual tool implementations.

## AbstractReplaceStringTool Base Class

The AbstractReplaceStringTool serves as a foundational base class for text modification tools in the vscode-copilot-chat extension. This abstract class implements the ICopilotTool interface and provides a comprehensive framework for handling string replacement operations across files, with built-in support for error handling, telemetry, and edit validation.

The class is designed with dependency injection, accepting numerous services through its constructor parameters, including configuration, workspace, notebook, file system, and telemetry services. This design promotes loose coupling and makes the class easily testable. The tool operates on the IAbstractReplaceStringInput interface, which defines the basic structure for replacement operations with filePath, oldString, and newString properties.

Key lifecycle methods include:
- **invoke**: The main entry point that orchestrates the tool execution
- **prepareEdits**: Prepares and validates edits before application
- **applyAllEdits**: Applies the prepared edits to the workspace
- **generateEdit**: Performs the actual string replacement logic
- **resolveInput**: Resolves tool input with context information

The class implements a caching mechanism for ReplaceStringsOperation instances to optimize performance when the same input is processed multiple times. It also includes sophisticated error handling with specific error types like NoMatchError, MultipleMatchesError, and NoChangeError, each providing detailed context for troubleshooting.

The abstract methods that must be implemented by subclasses are:
- **toolName()**: Returns the specific tool name identifier
- **extractReplaceInputs()**: Extracts one or more IAbstractReplaceStringInput objects from the tool's input type

This base class handles the complexity of text replacement operations, including handling different line ending formats (CRLF vs LF), validating file existence, and managing notebook document edits. It also integrates with the edit survival tracking system to monitor how long AI-generated edits persist in the codebase.

```mermaid
classDiagram
class AbstractReplaceStringTool {
+_promptContext IBuildPromptContext
+lastOperation Promise~IPrepareEdit[]~
+invoke(options, token) Promise~LanguageModelToolResult~
+prepareEdits(options, token) Promise~IPrepareEdit[]~
+applyAllEdits(options, edits, token) Promise~ExtendedLanguageModelToolResult~
+resolveInput(input, promptContext) Promise~T~
+prepareInvocation(options, token) Promise~PreparedToolInvocation~
-_prepareEdits(options, input, token) Promise~IPrepareEdit[]~
-_prepareEditsForFile(options, input, token) Promise~IPrepareEdit~
-generateEdit(uri, document, options, input, didHealRef, token) Promise~{edits, updatedFile}~
-sendReplaceTelemetry(outcome, options, input, file, isNotebookDocument, didHeal) Promise~void~
-sendHealingTelemetry(options, healError, applicationError) Promise~void~
-modelForTelemetry(options) Promise~string~
-modelObjectForTelemetry(options) string
-recordEditSuccess(options, success) Promise~void~
-_errorConflictingEdits(results) void
-generateConfirmationDetails(replaceInputs, urisNeedingConfirmation, options, token) Promise~string~
}
class ICopilotTool {
<<interface>>
+invoke? Invoke method
+prepareInvocation? Prepare invocation method
+filterEdits?(resource) Promise~IEditFilterData | undefined~
+provideInput?(promptContext) Promise~T | undefined~
+resolveInput?(input, promptContext, mode) Promise~T~
+alternativeDefinition?(tool, endpoint) LanguageModelToolInformation
}
AbstractReplaceStringTool --|> ICopilotTool
AbstractReplaceStringTool : Uses IConfigurationService
AbstractReplaceStringTool : Uses IWorkspaceService
AbstractReplaceStringTool : Uses IToolsService
AbstractReplaceStringTool : Uses INotebookService
AbstractReplaceStringTool : Uses IFileSystemService
AbstractReplaceStringTool : Uses ITelemetryService
AbstractReplaceStringTool : Uses IEndpointProvider
AbstractReplaceStringTool : Uses IEditToolLearningService
AbstractReplaceStringTool : Uses ILogService
**Diagram sources**
- [abstractReplaceStringTool.tsx](file : //src/extension/tools/node/abstractReplaceStringTool.tsx#L63-L581)
**Section sources**
- [abstractReplaceStringTool.tsx](file : //src/extension/tools/node/abstractReplaceStringTool.tsx#L63-L581)
## ToolsService Registry and Execution Manager
The toolsService serves as the central registry and execution manager for all tools in the vscode-copilot-chat extension. It provides a unified interface for tool discovery, invocation, and lifecycle management, acting as the bridge between the extension's tool implementations and the VS Code language model API.
The service is implemented as an abstract BaseToolsService class that provides common functionality, with concrete implementations for different environments. The core interface IToolsService defines the essential operations :
- **tools** : Provides access to all registered LanguageModelToolInformation objects
- **copilotTools** : Contains the tool implementations from this extension
- **invokeTool** : Executes a specific tool with given options
- **getTool** : Retrieves a specific tool by name
- **validateToolInput** : Validates input against a tool's schema
- **getEnabledTools** : Determines which tools should be enabled for a given request
The service uses a lazy initialization pattern with the Lazy class to defer the creation of tool instances until they are actually needed, improving startup performance. It maintains a registry of tools through the ToolRegistry singleton, which collects all registered tool constructors and extensions.
Key features of the toolsService include :
- **Schema validation** : Uses AJV (Another JSON Schema Validator) to validate tool inputs against their JSON schemas
- **Input normalization** : Handles cases where JSON strings are provided where objects are expected
- **Telemetry integration** : Tracks tool invocations and validation errors
- **Caching** : Implements an LRUCache for compiled JSON schemas to avoid recompilation
- **Event system** : Emits onWillInvokeTool events to allow subscribers to react to tool invocations
The service also handles the mapping of contributed tool names, normalizing tool names from external contributions to the internal naming convention. This allows the extension to work seamlessly with tools contributed by other extensions while maintaining internal consistency.
```mermaid
classDiagram
    class IToolsService {
        <<interface>>
        +_serviceBrand undefined
        +onWillInvokeTool Event~IOnWillInvokeToolEvent~
        +tools ReadonlyArray~LanguageModelToolInformation~
        +copilotTools ReadonlyMap~ToolName, ICopilotTool~unknown~~
        +getCopilotTool(name) ICopilotTool~unknown~ | undefined
        +invokeTool(name, options, token) Thenable~LanguageModelToolResult2~
        +getTool(name) LanguageModelToolInformation | undefined
        +getToolByToolReferenceName(name) LanguageModelToolInformation | undefined
        +validateToolInput(name, input) IToolValidationResult
        +validateToolName(name) string | undefined
        +getEnabledTools(request, endpoint, filter) LanguageModelToolInformation[]
    }
    
    class BaseToolsService {
        +_onWillInvokeTool Emitter~IOnWillInvokeToolEvent~
        +onWillInvokeTool Event~IOnWillInvokeToolEvent~
        +ajv Ajv
        +didWarnAboutValidationError Set~string~ | undefined
        +schemaCache LRUCache~ValidateFunction~
        +validateToolInput(name, input) IToolValidationResult
        +validateToolName(name) string | undefined
        -ajvValidateForTool(toolName, fn, inputObj) IToolValidationResult
        -getObjectPropertyByPath(obj, jsonPointerPath) {parent, propertyName} | null
    }
    
    class ToolsService {
        +_copilotTools Lazy~Map~ToolName, ICopilotTool~unknown~~~
        +_toolExtensions Lazy~Map~ToolName, ICopilotToolExtension~unknown~~~
        +_contributedToolCache {input, output}
        +tools ReadonlyArray~LanguageModelToolInformation~
        +copilotTools Map~ToolName, ICopilotTool~unknown~~
        +invokeTool(name, options, token) Thenable~LanguageModelToolResult | LanguageModelToolResult2~
        +getCopilotTool(name) ICopilotTool~unknown~ | undefined
        +getTool(name) LanguageModelToolInformation | undefined
        +getToolByToolReferenceName(name) LanguageModelToolInformation | undefined
        +getEnabledTools(request, endpoint, filter) LanguageModelToolInformation[]
    }
    
    class NullToolsService {
        +_serviceBrand undefined
        +tools readonly LanguageModelToolInformation[]
        +copilotTools Map
        +invokeTool(id, options, token) Promise~LanguageModelToolResult2~
        +getTool(id) LanguageModelToolInformation | undefined
        +getCopilotTool(name) ICopilotTool~unknown~ | undefined
        +getToolByToolReferenceName(name) LanguageModelToolInformation | undefined
        +getEnabledTools() LanguageModelToolInformation[]
    }
    
    IToolsService <|-- BaseToolsService
    BaseToolsService <|-- ToolsService
    BaseToolsService <|-- NullToolsService
    ToolsService : Uses ILogService
    ToolsService : Uses IInstantiationService

**Diagram sources**
- [toolsService.ts](file://src/extension/tools/common/toolsService.ts#L47-L253)
- [toolsService.ts](file://src/extension/tools/vscode-node/toolsService.ts#L16-L154)

**Section sources**
- [toolsService.ts](file://src/extension/tools/common/toolsService.ts#L47-L253)
- [toolsService.ts](file://src/extension/tools/vscode-node/toolsService.ts#L16-L154)

## Tool Utilities

The toolUtils module provides a collection of utility functions that are used across various tools in the vscode-copilot-chat extension. These utilities handle common operations such as string manipulation, path handling, and error formatting, promoting code reuse and consistency across tool implementations.

The primary functions in this module include:

- **checkCancellation**: A utility function that checks if a CancellationToken has been triggered and throws a CancellationError if so. This allows tools to gracefully handle cancellation requests from the user or system.

- **toolTSX**: A helper function that renders a PromptPiece using the prompt-tsx framework and returns it as a LanguageModelToolResult. This simplifies the process of returning structured content from tools.

- **inputGlobToPattern**: Converts a user input glob or file path into a VS Code glob pattern or RelativePattern. This function includes a model-specific workaround for GPT-4.1, which struggles with appending '/**' to patterns, demonstrating how the utilities adapt to specific model behaviors.

- **resolveToolInputPath**: Resolves a tool input path string to a URI using the IPromptPathRepresentationService. This ensures consistent path resolution across tools and handles invalid paths by throwing descriptive errors.

- **assertFileOkForTool**: Validates that a file can be used by a tool, checking that it's not excluded by ignore rules and that it's either within the workspace or open in an editor. This function uses dependency injection to access the necessary services.

- **assertFileNotContentExcluded**: Specifically checks if a file is configured to be ignored by Copilot through the IIgnoreService, throwing an error if the file should be excluded.

These utilities follow a functional programming approach, with pure functions that have no side effects and clear input/output contracts. They leverage dependency injection through the ServicesAccessor pattern, allowing them to access necessary services without tight coupling to specific implementations.

The module also demonstrates good error handling practices, with descriptive error messages that help users understand what went wrong and how to fix it. For example, when a file path is invalid, the error message specifically mentions that an absolute path should be used.

```mermaid
classDiagram
class toolUtils {
+checkCancellation(token) void
+toolTSX(insta, options, piece, token) Promise~LanguageModelToolResult~
+inputGlobToPattern(query, workspaceService, modelFamily) GlobPattern[]
+resolveToolInputPath(path, promptPathRepresentationService) URI
+isFileOkForTool(accessor, uri) Promise~boolean~
+assertFileOkForTool(accessor, uri) Promise~void~
+assertFileNotContentExcluded(accessor, uri) Promise~void~
}
toolUtils : Uses ICustomInstructionsService
toolUtils : Uses IIgnoreService
toolUtils : Uses IPromptPathRepresentationService
toolUtils : Uses ITabsAndEditorsService
toolUtils : Uses IWorkspaceService
toolUtils : Uses IInstantiationService
**Diagram sources**
- [toolUtils.ts](file : //src/extension/tools/common/toolUtils.ts#L24-L125)
**Section sources**
- [toolUtils.ts](file : //src/extension/tools/common/toolUtils.ts#L24-L125)
## Edit File Tool Utilities
The editFileToolUtils module provides specialized utilities for file operations, including reading, writing, and conflict resolution. These utilities are specifically designed to support text modification tools and handle the complexities of file editing in a collaborative environment.
Key components of this module include :
- **Error classes** : A hierarchy of error classes that provide specific error types for different failure modes :
- EditError : Base class for all edit-related errors
- NoMatchError : Thrown when no match is found for a string replacement
- MultipleMatchesError : Thrown when multiple matches are found
- NoChangeError : Thrown when an edit would result in no changes
- ContentFormatError : Thrown when there are issues with content format
- **formatDiffAsUnified** : Formats a diff computed by IDiffService as a unified diff string, which can be displayed to users to show the changes that would be made. This function handles the conversion of line changes to the standard diff format with '+' for additions, '-' for removals, and ' ' for context lines.
- **findAndReplaceOne** : The core function for performing string replacements with multiple matching strategies :
- Exact match : Fastest strategy that looks for an exact string match
- Whitespace-flexible match : Handles cases where whitespace differs between the search string and target
- Fuzzy match : Uses regex patterns to match with flexible whitespace and newline formats
- Similarity-based match : Last resort that uses Levenshtein distance to find similar text
- **applyEdit** : The main function that applies edits to files, coordinating between the workspace API, notebook services, and text document snapshots. It handles both text documents and notebooks, and manages the creation of TextEdit objects that represent the changes.
- **Path safety validation** : Functions that validate paths to prevent security issues, particularly on Windows systems. This includes checking for null bytes, NTFS alternate data streams, invalid characters, and reserved device names.
- **Confirmation system** : Utilities for determining when user confirmation is required before applying edits, based on file location, permissions, and sensitivity. This includes checking for edits outside the workspace, to system files, or to sensitive locations.
The module implements a sophisticated matching strategy that tries different approaches in order of reliability, falling back to less precise methods only when necessary. This ensures that edits are applied accurately while still being able to handle cases where the model's output doesn't perfectly match the target text.
```mermaid
classDiagram
    class EditError {
        +message string
        +kindForTelemetry string
        +constructor(message, kindForTelemetry)
    }
    
    class NoMatchError {
        +file string
        +constructor(message, file)
    }
    
    class MultipleMatchesError {
        +file string
        +constructor(message, file)
    }
    
    class NoChangeError {
        +file string
        +constructor(message, file)
    }
    
    class ContentFormatError {
        +file string
        +constructor(message, file)
    }
    
    class editFileToolUtils {
        +formatDiffAsUnified(accessor, uri, oldContent, newContent) Promise~string~
        +calculateSimilarity(str1, str2) number
        +findAndReplaceOne(text, oldStr, newStr, eol) MatchResult
        +tryExactMatch(text, oldStr, newStr) MatchResult
        +tryWhitespaceFlexibleMatch(text, oldStr, newStr, eol) MatchResult
        +tryFuzzyMatch(text, oldStr, newStr, eol) MatchResult
        +trySimilarityMatch(text, oldStr, newStr, eol, threshold) MatchResult
        +applyEdit(uri, old_string, new_string, workspaceService, notebookService, alternativeNotebookContent, languageModel) Promise~{patch, updatedFile, edits}~
        +assertPathIsSafe(fsPath, _isWindows) void
        +makeUriConfirmationChecker(configuration, workspaceService, customInstructionsService) Function
        +createEditConfirmation(accessor, uris, detailMessage) Promise~PreparedToolInvocation~
        +canExistingFileBeEdited(accessor, uri) Promise~boolean~
        +logEditToolResult(logService, requestId, ...opts) void
        +openDocumentAndSnapshot(accessor, promptContext, uri) Promise~NotebookDocumentSnapshot | TextDocumentSnapshot~
    }
    
    NoMatchError --|> EditError
    MultipleMatchesError --|> EditError
    NoChangeError --|> EditError
    ContentFormatError --|> EditError

**Diagram sources**
- [editFileToolUtils.tsx](file://src/extension/tools/node/editFileToolUtils.tsx#L49-L960)

**Section sources**
- [editFileToolUtils.tsx](file://src/extension/tools/node/editFileToolUtils.tsx#L49-L960)

## Concrete Tool Implementations

The vscode-copilot-chat extension demonstrates the practical application of the base classes and utilities through concrete tool implementations. Two primary examples illustrate how the AbstractReplaceStringTool base class is extended to create specialized tools for different use cases.

### ReplaceStringTool

The ReplaceStringTool is a direct extension of AbstractReplaceStringTool that handles single-file string replacements. It implements the required abstract methods:

- **toolName()**: Returns ToolName.ReplaceString, identifying this tool in the registry
- **extractReplaceInputs()**: Extracts a single IAbstractReplaceStringInput from the tool's input parameters, which include explanation, filePath, oldString, and newString

This tool demonstrates the minimal implementation required to create a functional text modification tool. By extending the base class, it inherits all the sophisticated error handling, telemetry, and edit validation logic while only needing to specify the tool-specific behavior.

### MultiReplaceStringTool

The MultiReplaceStringTool extends AbstractReplaceStringTool to handle multiple file replacements in a single operation. Its key implementation details include:

- **extractReplaceInputs()**: Maps an array of replacement objects to multiple IAbstractReplaceStringInput instances, enabling batch operations across multiple files
- **invoke()**: Includes additional telemetry to track the success and failure rates of individual replacements, providing valuable insights into tool effectiveness
- **Edit merging**: Implements logic to merge successful edits to the same file, ensuring that positions aren't clobbered when multiple replacements target the same file

This tool demonstrates how the base class can be extended to support more complex scenarios while maintaining consistency with the overall architecture. The implementation includes sophisticated error handling that continues processing remaining replacements even when some fail, providing partial success when possible.

Both tools use the ToolRegistry to register themselves, making them available through the toolsService. This registration pattern ensures that all tools are discoverable and can be invoked through the standard interface.

```mermaid
classDiagram
    class AbstractReplaceStringTool {
        <<abstract>>
        +invoke(options, token) Promise~LanguageModelToolResult~
        +prepareEdits(options, token) Promise~IPrepareEdit[]~
        +applyAllEdits(options, edits, token) Promise~ExtendedLanguageModelToolResult~
        +resolveInput(input, promptContext) Promise~T~
        +prepareInvocation(options, token) Promise~PreparedToolInvocation~
        -toolName() ToolName
        -extractReplaceInputs(input) IAbstractReplaceStringInput[]
    }
    
    class ReplaceStringTool {
        +static toolName ToolName
        +invoke(options, token) Promise~LanguageModelToolResult~
        -toolName() ToolName
        -extractReplaceInputs(input) IAbstractReplaceStringInput[]
    }
    
    class MultiReplaceStringTool {
        +static toolName ToolName
        +invoke(options, token) Promise~LanguageModelToolResult~
        -toolName() ToolName
        -extractReplaceInputs(input) IAbstractReplaceStringInput[]
    }
    
    AbstractReplaceStringTool <|-- ReplaceStringTool
    AbstractReplaceStringTool <|-- MultiReplaceStringTool
    ReplaceStringTool : Registers with ToolRegistry
    MultiReplaceStringTool : Registers with ToolRegistry

**Diagram sources**
- [replaceStringTool.tsx](file://src/extension/tools/node/replaceStringTool.tsx#L18-L40)
- [multiReplaceStringTool.tsx](file://src/extension/tools/node/multiReplaceStringTool.tsx#L20-L113)

**Section sources**
- [replaceStringTool.tsx](file://src/extension/tools/node/replaceStringTool.tsx#L18-L40)
- [multiReplaceStringTool.tsx](file://src/extension/tools/node/multiReplaceStringTool.tsx#L20-L113)

## Best Practices for Extension and Usage

Based on the analysis of the vscode-copilot-chat extension, several best practices emerge for extending base classes and leveraging utilities to maintain consistency and reduce code duplication:

### Extending Base Classes

1. **Implement required abstract methods**: When extending AbstractReplaceStringTool, always implement the toolName() and extractReplaceInputs() methods to provide tool-specific behavior.

2. **Leverage inheritance**: Take advantage of the comprehensive functionality provided by base classes rather than reimplementing common features like error handling, telemetry, and edit validation.

3. **Follow naming conventions**: Use the ToolRegistry and ToolName enum to ensure consistent tool naming across the extension.

4. **Register tools properly**: Always register new tools with the ToolRegistry to make them available through the toolsService.

### Leveraging Utilities

1. **Use shared error handling**: Utilize the standardized error classes (NoMatchError, MultipleMatchesError, etc.) to provide consistent error messages and telemetry.

2. **Apply path validation**: Use resolveToolInputPath and assertFileOkForTool to ensure safe and valid file operations.

3. **Implement proper cancellation handling**: Use checkCancellation to gracefully handle cancellation requests.

4. **Leverage matching strategies**: Use the findAndReplaceOne function with its multiple matching strategies rather than implementing custom string matching logic.

5. **Use confirmation workflows**: Implement the createEditConfirmation pattern to ensure users are aware of sensitive file modifications.

### Design Principles

1. **Dependency injection**: Follow the pattern of injecting services through constructors rather than accessing them globally, which improves testability and maintainability.

2. **Functional utilities**: Prefer pure functions in utility modules that have no side effects and clear input/output contracts.

3. **Progressive enhancement**: Implement features with multiple strategies, starting with the most reliable and falling back to less precise methods when necessary.

4. **Comprehensive telemetry**: Include detailed telemetry for all tool operations to enable monitoring and improvement of tool effectiveness.

5. **Security considerations**: Always validate paths and check permissions before performing file operations, especially on Windows systems.

These practices ensure that new tools integrate seamlessly with the existing architecture, maintain consistency in behavior and user experience, and benefit from the robust error handling and validation already implemented in the base components.

**Section sources**
- [abstractReplaceStringTool.tsx](file://src/extension/tools/node/abstractReplaceStringTool.tsx#L63-L581)
- [toolsService.ts](file://src/extension/tools/common/toolsService.ts#L47-L253)
- [toolUtils.ts](file://src/extension/tools/common/toolUtils.ts#L24-L125)
- [editFileToolUtils.tsx](file://src/extension/tools/node/editFileToolUtils.tsx#L49-L960)

## Conclusion

The vscode-copilot-chat extension demonstrates a well-architected system for tool implementation with a clear separation of concerns between base classes, utilities, and concrete implementations. The AbstractReplaceStringTool provides a robust foundation for text modification tools, handling complex operations like string matching, error handling, and telemetry while allowing subclasses to focus on tool-specific behavior.

The toolsService acts as a central registry and execution manager, providing a unified interface for tool discovery and invocation while handling schema validation and input normalization. This service-oriented architecture promotes loose coupling and makes the system easily extensible.

The utility modules provide shared functionality for common operations, reducing code duplication and ensuring consistency across tools. The editFileToolUtils in particular demonstrates sophisticated handling of file operations with multiple matching strategies and comprehensive error handling.

The concrete implementations of ReplaceStringTool and MultiReplaceStringTool show how the base classes can be extended to create specialized tools for different use cases while maintaining architectural consistency. The use of the ToolRegistry ensures that all tools are properly registered and discoverable.

Overall, this architecture exemplifies best practices in software design, including dependency injection, separation of concerns, and progressive enhancement. By following the patterns demonstrated in this codebase, developers can create new tools that integrate seamlessly with the existing system while benefiting from the robust infrastructure already in place.

**Section sources**
- [abstractReplaceStringTool.tsx](file://src/extension/tools/node/abstractReplaceStringTool.tsx#L63-L581)
- [toolsService.ts](file://src/extension/tools/common/toolsService.ts#L47-L253)
- [toolUtils.ts](file://src/extension/tools/common/toolUtils.ts#L24-L125)
- [editFileToolUtils.tsx](file://src/extension/tools/node/editFileToolUtils.tsx#L49-L960)