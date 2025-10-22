# PR Review Comment Fix Strategies

Quick reference for implementing common GitHub PR review comment fixes with standardized patterns.

## Quick Fix Commands

### Single Line Changes
```bash
# Replace line 123 with new content
sed -i '123s/.*/new content/' filename.py

# Pattern replacement
sed -i 's/old_pattern/new_pattern/g' filename.js

# Insert before line number
sed -i '50i\new line to insert' filename.py

# Insert after line number
sed -i '50a\new line to insert' filename.py
```

### Multi-line Changes
```bash
# Replace lines 10-15 with new content
sed -i '10,15c\
new line 1\
new line 2' filename.py

# Using heredoc for complex blocks
cat > temp_block.txt << 'EOF'
new multi-line content
goes here
EOF
sed -i '10,15d' filename.py
sed -i '9r temp_block.txt' filename.py
rm temp_block.txt
```

## Language-Specific Fix Patterns

### Python
| Issue | Fix Pattern | Example |
|-------|-------------|---------|
| Type hints | Add type annotations | `def func(param: str) -> bool:` |
| Docstrings | Add function documentation | See docstring template below |
| Import ordering | stdlib first, then third-party | `import os` then `import requests` |
| String formatting | Use f-strings | `f"value = {variable}"` |
| Boolean logic | Clean comparisons | `if condition:` not `if condition == True:` |

```python
# Docstring template
def process_data(data: List[str]) -> Dict[str, int]:
    """
    Process input data and return aggregated counts.

    Args:
        data: List of strings to process

    Returns:
        Dictionary with aggregated counts

    Raises:
        ValueError: If data contains invalid items
    """
```

### JavaScript/TypeScript
| Issue | Fix Pattern | Example |
|-------|-------------|---------|
| Variable declaration | Use const/let instead of var | `const value = 5` |
| Functions | Use arrow functions | `const func = () => {}` |
| Object access | Use destructuring | `const {prop} = object` |
| Strings | Use template literals | `` `value = ${variable}` `` |

### YAML/Configuration
| Issue | Fix Pattern | Example |
|-------|-------------|---------|
| Indentation | Use 2 spaces consistently | `key: value` |
| Quoting | Quote special characters | `"special: value"` |
| Arrays | Alphabetical when appropriate | Sort by name/ID |
| Duplicates | Remove duplicate keys | Use unique keys only |

### Dockerfile
| Issue | Fix Pattern | Example |
|-------|-------------|---------|
| RUN commands | Consolidate with && | `RUN apt-get update && apt-get install` |
| Versions | Use specific tags | `FROM python:3.11-slim` |
| Caches | Remove package caches | `apt-get clean && rm -rf /var/lib/apt` |
| Health checks | Add HEALTHCHECK | `HEALTHCHECK --interval=30s` |

## File Type Fix Strategies

### Source Code Files
1. **Parse comment location** from file path and line number
2. **Apply appropriate pattern** for language and issue type
3. **Validate syntax** with linter or compiler
4. **Test functionality** if tests available

### Configuration Files
1. **Validate syntax** before and after changes
2. **Check environment compatibility**
3. **Verify no breaking changes** to existing functionality
4. **Update documentation** if behavior changes

### Documentation Files
1. **Maintain formatting consistency** with project style
2. **Verify accuracy** of technical content
3. **Check link validity** for internal/external references
4. **Update examples** if code changes affect them

## Automated Tool Fixes

### Linter Categories
| Category | Common Fixes | Validation |
|----------|--------------|------------|
| Unused imports | Remove unused statements | Run import linter |
| Missing error handling | Add try/catch blocks | Test error paths |
| Style violations | Apply style fixes | Re-run linter |
| Security vulnerabilities | Sanitize inputs, add validation | Security scan |

### Security Fixes
| Issue | Fix Pattern | Example |
|-------|-------------|---------|
| Input validation | Add validation checks | `if not isinstance(data, str):` |
| SQL injection | Use parameterized queries | `cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))` |
| XSS prevention | Escape HTML output | `escape_html(user_input)` |
| Auth issues | Add authentication checks | `@login_required` decorator |

## Quality Validation

### Pre-Commit Checklist
- [ ] Syntax check passes (compiler/linter)
- [ ] No new linting errors introduced
- [ ] Tests still pass (if applicable)
- [ ] Fix addresses original comment concern
- [ ] No regression in functionality

### Git Operations
```bash
# Stage specific fix
git add filename.ext

# Commit with comment reference
git commit -m "Fix: Address PR review comment

- Implement suggested change for [specific issue]
- Maintains compatibility with existing functionality
- Resolves thread: [thread_id]"

# Optional: branch for complex fixes
git checkout -b fix/pr-comment-[number]
git commit -m "Address review comment #[number]"
git push origin fix/pr-comment-[number]
```

## Common Anti-Patterns to Avoid

| Anti-Pattern | Why Bad | Better Approach |
|--------------|---------|-----------------|
| Silent failures | Hides problems | Add proper error handling |
| Breaking changes | Risks compatibility | Maintain backward compatibility |
| Over-engineering | Adds complexity | Use simplest solution |
| Style inconsistencies | Reduces readability | Follow project conventions |
| Missing tests | Allows regressions | Add test coverage for fixes |

## Quick Reference Templates

### Error Handling Addition
```python
try:
    risky_operation()
except SpecificException as e:
    logger.error(f"Operation failed: {e}")
    raise
```

### Performance Optimization
```python
# Instead of O(n²) nested loops
result = [item for item in large_list if condition(item)]

# Instead of repeated string concatenation
result = ''.join(list_of_strings)
```

### Boolean Logic Cleanup
```python
# Bad
if not condition is True:
if x == True:

# Good
if condition is not True:
if x:
```

## Fix Implementation Workflow

1. **Identify Target**: Parse file path and line number from comment
2. **Select Strategy**: Choose appropriate fix pattern based on issue type
3. **Apply Fix**: Implement change using language-specific pattern
4. **Validate**: Check syntax, run tests, verify no regressions
5. **Resolve**: Mark review thread as resolved when fix confirmed