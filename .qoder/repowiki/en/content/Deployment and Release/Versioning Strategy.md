# Versioning Strategy

<cite>
**Referenced Files in This Document**   
- [package.json](file://package.json)
- [package.nls.json](file://package.nls.json)
- [CHANGELOG.md](file://CHANGELOG.md)
- [vscode.proposed.chatProvider.d.ts](file://src/extension/vscode.proposed.chatProvider.d.ts)
- [vscode.proposed.chatSessionsProvider.d.ts](file://src/extension/vscode.proposed.chatSessionsProvider.d.ts)
- [vscode.proposed.defaultChatParticipant.d.ts](file://src/extension/vscode.proposed.defaultChatParticipant.d.ts)
- [vscode.proposed.chatEditing.d.ts](file://src/extension/vscode.proposed.chatEditing.d.ts)
- [vscode.proposed.chatStatusItem.d.ts](file://src/extension/vscode.proposed.chatStatusItem.d.ts)
</cite>

## Table of Contents
1. [Semantic Versioning Scheme](#semantic-versioning-scheme)
2. [Proposed API Dependencies and Versioning](#proposed-api-dependencies-and-versioning)
3. [Package Metadata Updates](#package-metadata-updates)
4. [Breaking Changes and Migration Guidance](#breaking-changes-and-migration-guidance)
5. [Backward Compatibility and Deprecation Policies](#backward-compatibility-and-deprecation-policies)
6. [VS Code Version Support](#vs-code-version-support)

## Semantic Versioning Scheme

The vscode-copilot-chat extension follows semantic versioning (SemVer) with a three-part version number (MAJOR.MINOR.PATCH) as defined in the package.json file. The current version is 0.34.0, indicating the extension is in active development with regular updates.

**Major versions** are incremented for significant changes that may introduce breaking changes to the API, user interface, or core functionality. These releases typically include substantial new features, architectural changes, or major refactoring efforts that could affect extension consumers or users.

**Minor versions** are incremented for backward-compatible feature additions and improvements. These releases introduce new capabilities, enhance existing functionality, or add support for new VS Code APIs while maintaining compatibility with previous versions. For example, the 0.32 release introduced fully qualified tool names and improved edit tools for bring-your-own-key models.

**Patch versions** are incremented for backward-compatible bug fixes, security patches, and minor improvements that do not add new features. These releases address issues discovered in previous versions without changing the public API or introducing new functionality.

The versioning scheme is reflected in the CHANGELOG.md file, which documents changes for each release with clear version numbers and release dates. The extension maintains a regular release cadence, with updates typically released monthly to incorporate new features, improvements, and bug fixes.

**Section sources**
- [package.json](file://package.json#L5)
- [CHANGELOG.md](file://CHANGELOG.md#L1)

## Proposed API Dependencies and Versioning

The vscode-copilot-chat extension relies heavily on proposed VS Code APIs (vscode.proposed.*.d.ts) that are marked as experimental and subject to change. These proposed APIs are critical to the extension's functionality and influence versioning decisions significantly.

The extension consumes several proposed APIs, each with its own version number indicated in comments at the top of the file. For example:
- `vscode.proposed.chatProvider.d.ts` (version: 4)
- `vscode.proposed.chatSessionsProvider.d.ts` (version: 3)
- `vscode.proposed.defaultChatParticipant.d.ts` (version: 4)

When VS Code updates these proposed APIs, the extension must adapt to the changes, which may require breaking changes in the extension itself. The versioning strategy accounts for this dependency by:
1. Monitoring VS Code API changes and planning extension updates accordingly
2. Incrementing the minor version when adapting to non-breaking API changes
3. Incrementing the major version when VS Code API changes require breaking changes in the extension
4. Maintaining backward compatibility with multiple versions of proposed APIs when possible

The extension's package.json file includes an `enabledApiProposals` array that explicitly lists the proposed APIs the extension depends on. This serves as a contract with VS Code, ensuring the extension only activates when the required proposed APIs are available. Changes to this list may trigger version increments:
- Adding new proposed APIs typically results in a minor version increment
- Removing deprecated proposed APIs may require a major version increment if it represents a significant functionality change

The extension team closely coordinates with the VS Code team to anticipate API changes and plan versioning accordingly, ensuring a smooth transition for users when proposed APIs evolve or become stable.

```mermaid
classDiagram
class Extension {
+version : string
+enabledApiProposals : string[]
+activationEvents : string[]
}
class ProposedAPI {
+name : string
+version : number
+status : "proposed" | "stable"
}
class VSCode {
+version : string
+apiProposals : Map<string, ProposedAPI>
}
Extension --> ProposedAPI : "depends on"
ProposedAPI --> VSCode : "provided by"
Extension --> VSCode : "requires"
note right of Extension
Version increments based on
API dependency changes :
- Major : Breaking API changes
- Minor : New API additions
- Patch : API bug fixes
end note
```

**Diagram sources **
- [package.json](file://package.json#L91-L140)
- [vscode.proposed.chatProvider.d.ts](file://src/extension/vscode.proposed.chatProvider.d.ts#L6)
- [vscode.proposed.chatSessionsProvider.d.ts](file://src/extension/vscode.proposed.chatSessionsProvider.d.ts#L6)

**Section sources**
- [package.json](file://package.json#L91-L140)
- [vscode.proposed.chatProvider.d.ts](file://src/extension/vscode.proposed.chatProvider.d.ts)
- [vscode.proposed.chatSessionsProvider.d.ts](file://src/extension/vscode.proposed.chatSessionsProvider.d.ts)

## Package Metadata Updates

During version increments, the extension updates both package.json and localized package metadata (package.nls.json) to reflect changes and provide accurate information to users.

In package.json, the following fields are updated during version increments:
- `version`: Updated to reflect the new SemVer version
- `build`: Incremented for each build within the same version
- `completionsCoreVersion`: Updated when the core completions library changes
- `engines.vscode`: Updated to reflect the minimum VS Code version required
- `badges`: Updated to reflect current status and links

The package.nls.json file contains localized strings used throughout the extension and is updated to reflect new features, commands, and UI elements introduced in each release. When new functionality is added, corresponding entries are added to package.nls.json with appropriate keys and values. For example, new command entries follow the pattern:
```
"github.copilot.command.[commandName]": "[Localized command label]"
```

When deprecating features, the team follows a process of:
1. Marking the feature as deprecated in documentation
2. Maintaining the package.nls.json entry with a deprecated indicator
3. Eventually removing the entry in a future major version

The version increment process includes automated checks to ensure:
- All new UI elements have corresponding localization entries
- Deprecated entries are properly marked
- Version numbers are consistent across files
- Metadata accurately reflects the current state of the extension

**Section sources**
- [package.json](file://package.json)
- [package.nls.json](file://package.nls.json)

## Breaking Changes and Migration Guidance

Breaking changes are communicated through the CHANGELOG.md file with clear documentation of what has changed and how users should adapt. The extension follows a structured approach to introducing breaking changes:

1. **Deprecation Period**: Before removing or significantly changing functionality, the team marks it as deprecated for at least one minor version cycle, providing warnings in the console and documentation.

2. **Clear Documentation**: Breaking changes are documented in the CHANGELOG with:
   - A clear description of what changed
   - The rationale for the change
   - Step-by-step migration instructions
   - Examples of old vs. new usage patterns

3. **Migration Guidance**: For API changes, the team provides:
   - Code examples showing the migration path
   - Automated codemods when possible
   - Links to updated documentation
   - Support channels for assistance

4. **Version Signaling**: Breaking changes are always accompanied by a major version increment, signaling to users and dependent systems that compatibility may be affected.

The CHANGELOG.md file serves as the primary communication channel for breaking changes, with each release section clearly indicating any breaking changes and providing migration guidance. For example, the 0.32 release documented the introduction of fully qualified tool names and included a code action to help migrate from the old notation.

The team also uses VS Code's built-in deprecation mechanisms, such as marking APIs as `@deprecated` in TypeScript definitions and providing alternative approaches in documentation.

**Section sources**
- [CHANGELOG.md](file://CHANGELOG.md)

## Backward Compatibility and Deprecation Policies

The vscode-copilot-chat extension maintains a strong commitment to backward compatibility while evolving its functionality. The deprecation policy balances innovation with stability:

**Deprecation Timeline**:
- **Announcement**: Features marked as deprecated with clear messaging
- **Maintenance Period**: Deprecated features continue to work for at least two minor versions
- **Removal**: Deprecated features are removed in a subsequent major version

**Compatibility Guarantees**:
- Patch versions are always backward compatible
- Minor versions maintain backward compatibility with public APIs
- Major versions may introduce breaking changes but provide migration paths

**Deprecation Process**:
1. Add deprecation notices to documentation and console output
2. Update TypeScript definitions with `@deprecated` tags
3. Provide alternative approaches and migration guidance
4. Monitor usage metrics to assess impact
5. Remove after sufficient notice period

The extension also maintains compatibility with multiple VS Code versions by:
- Testing against current and previous VS Code releases
- Using feature detection for newer APIs
- Providing fallbacks for older VS Code versions
- Clearly documenting minimum required VS Code version

This approach ensures users can upgrade the extension with confidence, knowing that their existing workflows will continue to function while they adapt to new patterns.

**Section sources**
- [CHANGELOG.md](file://CHANGELOG.md)
- [package.json](file://package.json#L26)

## VS Code Version Support

The extension supports a range of VS Code versions, with the minimum required version specified in the package.json file under `engines.vscode`. Currently, the extension requires VS Code version ^1.107.0-20251119, indicating compatibility with VS Code 1.107.0 and later versions.

The support policy includes:
- **Current Version**: Full feature support and regular updates
- **Previous Version**: Critical bug fixes and security patches
- **Older Versions**: Limited support, with users encouraged to upgrade

The extension uses VS Code's proposed API system to provide enhanced functionality when available, while maintaining core functionality on older versions. Feature detection is used to enable or disable functionality based on the VS Code version and available APIs.

The team monitors VS Code release cycles and plans extension updates to align with major VS Code releases, ensuring compatibility with the latest features while maintaining support for stable releases. Users are notified of required VS Code updates through the extension marketplace and in-product messaging.

**Section sources**
- [package.json](file://package.json#L26)