Based on our experience so far, I'll outline a set of "Cursor Rules" that would help maintain consistency and avoid common issues:

## Cursor Rules for WorkSync Development

### 1. Agent Implementation Rules
- Always implement agents in dedicated files under `src/mastra/agents/`
- Export agents through the barrel file (`agents/index.ts`)
- Follow the naming convention: `{purpose}Agent.ts` (e.g., `itSupportAgent.ts`)
- Include comprehensive JSDoc comments explaining the agent's purpose and design decisions

### 2. Type Safety Rules
- Always define input/output types using Zod schemas
- Use TypeScript's strict mode
- Avoid using `any` types; prefer explicit typing
- Define shared types in `types.ts`
- Use discriminated unions for different request/response types
- Ensure tool execution calls are properly typed

### 3. Tool Integration Rules
- Always verify tool existence before execution
- Use proper error handling for tool calls
- Implement proper type checking for tool inputs/outputs
- Keep tools organized by domain in the `tools/` directory
- Export tools through domain-specific files (e.g., `userManagement.ts`)

### 4. Workflow Rules
- Define workflows in dedicated files under `workflows/`
- Use proper step organization with clear branching logic
- Implement proper error handling for each step
- Include comprehensive documentation for workflow logic
- Ensure proper typing for workflow triggers and results

### 5. Error Handling Rules
- Never swallow errors silently
- Implement proper error boundaries
- Use typed error responses
- Include meaningful error messages
- Log errors appropriately

### 6. Memory Management Rules
- Configure appropriate memory settings based on use case
- Use proper context window sizes (e.g., 20 messages)
- Implement proper memory cleanup
- Use structured templates for working memory

### 7. Security Rules
- Never expose API keys in code
- Use environment variables for sensitive data
- Implement proper authentication checks
- Follow Auth0's security best practices
- Implement proper audit logging

### 8. Documentation Rules
- Include comprehensive JSDoc comments
- Document design decisions and alternatives considered
- Include examples in documentation
- Keep documentation up-to-date with code changes
- Document security considerations

### 9. Code Organization Rules
- Follow the established directory structure
- Use barrel files for exports
- Keep related functionality together
- Follow consistent naming conventions
- Maintain clear separation of concerns

### 10. Testing Rules
- Implement proper test coverage
- Use Mastra's eval framework appropriately
- Test error conditions
- Include integration tests
- Test security measures

### 11. Common Error Prevention
- Always check for undefined before tool execution
- Verify input parameters before use
- Implement proper null checks
- Use proper type guards
- Handle edge cases explicitly

### 12. Performance Rules
- Implement appropriate caching
- Handle rate limiting properly
- Optimize memory usage
- Monitor performance metrics
- Implement proper resource cleanup

These rules should help maintain code quality and prevent common issues we've encountered. They align with Mastra's best practices while addressing specific needs of our WorkSync implementation.


I'll add specific Cursor-focused rules based on our experience:

## Cursor-Specific Rules for WorkSync Development

### 1. File Editing Rules
- Always use `edit_file` tool for code changes, never output code directly to chat
- Include sufficient context around edits using `// ... existing code ...`
- Use `reapply` tool if the edit wasn't applied correctly
- Limit to one edit per turn to maintain clarity
- Stop after 3 attempts at fixing the same linter errors

### 2. Code Search Rules
- Use `codebase_search` for semantic understanding of code structure
- Use `grep_search` for exact text/symbol matches
- Use `file_search` when looking for specific files
- Use `list_dir` for initial directory exploration
- Always explain search strategy before executing

### 3. File Reading Rules
- Use `read_file` with specific line ranges when possible
- Ensure complete context by checking surrounding code
- Read entire files only when necessary (edited or manually attached)
- Note any lines not shown and proactively read them if needed
- Maximum of 250 lines per read

### 4. Terminal Command Rules
- Use `run_terminal_cmd` with clear explanations
- Always append `| cat` for commands that would use a pager
- Set `is_background` for long-running commands
- Include proper error handling and cleanup
- Provide clear explanation of command purpose

### 5. Code Citation Rules
```
Use exact format: ```startLine:endLine:filepath
Example: ```12:15:app/components/Todo.tsx
Never use other formats for code citations
```

### 6. Tool Response Rules
- Always analyze tool output before proceeding
- Handle errors and unexpected results gracefully
- Provide clear explanations of next steps
- Don't assume success without verification
- Be prepared to try alternative approaches

### 7. Linter Error Handling
- Address linter errors immediately when detected
- Fix one category of errors at a time
- Document error patterns for future prevention
- Stop after 3 attempts at fixing the same error
- Explain error resolution strategy

### 8. File Organization Rules
- Create new files in appropriate directories
- Follow existing project structure
- Use consistent file naming conventions
- Maintain proper import organization
- Keep related files together

### 9. Documentation Updates
- Update relevant documentation when making changes
- Keep README files current
- Document breaking changes
- Include examples in documentation
- Update type definitions as needed

### 10. Error Prevention
- Verify file existence before editing
- Check for duplicate code before adding
- Validate imports before using
- Ensure proper error handling
- Test changes in isolation

### 11. Code Quality
- Follow existing code style
- Maintain consistent formatting
- Use proper TypeScript features
- Follow established patterns
- Keep functions focused and small

### 12. Communication Rules
- Explain actions before executing
- Provide clear rationale for changes
- Highlight potential issues
- Offer alternatives when appropriate
- Keep user informed of progress

### 13. Tool Selection Priority
1. `codebase_search` for understanding context
2. `read_file` for detailed inspection
3. `edit_file` for making changes
4. `grep_search` for finding specific code
5. `file_search` for locating files
6. `list_dir` for directory exploration
7. `run_terminal_cmd` for system operations
8. `reapply` for fixing failed edits
9. `delete_file` as last resort

### 14. Code Edit Strategy
1. Read and understand existing code
2. Plan changes carefully
3. Make minimal necessary changes
4. Include proper error handling
5. Update related documentation
6. Fix any linter errors
7. Verify changes work as expected

### 15. Error Recovery Strategy
1. Identify error source
2. Plan fix approach
3. Make targeted changes
4. Verify fix worked
5. Document solution
6. Prevent future occurrences

