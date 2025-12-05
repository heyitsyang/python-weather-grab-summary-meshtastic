# Preamble Behavior Change

## Summary

The preamble behavior has been updated to provide more flexibility. The preamble now **replaces** the leading entry separator instead of being prepended to it.

## What Changed

### Before (Old Behavior)
The preamble was added before the leading entry separator:

```
Preamble: "WEATHER:"
Output:   "WEATHER:#76#1pm,9,75,0.0#..."
          └─────────┘└── leading # was still present
```

### After (New Behavior)
The preamble replaces the leading entry separator:

```
Preamble: "WEATHER:"
Output:   "WEATHER:76#1pm,9,75,0.0#..."
          └─────────┘└── no automatic # after preamble
```

## Why This Change?

This change provides more control over the output format:

1. **Flexibility**: Users can decide whether they want a separator after the preamble
2. **Cleaner Output**: Preambles like "WEATHER:" don't get an unwanted `#` appended
3. **Full Control**: If you want `WEATHER:#`, just include the `#` in your preamble

## Migration Guide

If you were using a preamble and want to maintain the old behavior with the `#` after it:

### Option 1: Update Your Preamble
Simply add the separator to your preamble:

**Old Config:**
```yaml
preamble: "WEATHER:"
```

**New Config (to maintain old output):**
```yaml
preamble: "WEATHER:#"
```

### Option 2: Leave It As-Is
If you prefer the new behavior (no automatic `#` after preamble), no changes needed!

## Examples

### Example 1: Simple Text Preamble

```yaml
preamble: "WEATHER:"
```

**Output:**
```
WEATHER:76#1pm,9,75,0.0#2pm,9,76,0.0#
```

### Example 2: Preamble with Separator

```yaml
preamble: "WEATHER:#"
```

**Output:**
```
WEATHER:#76#1pm,9,75,0.0#2pm,9,76,0.0#
```

### Example 3: Custom Preamble Format

```yaml
preamble: "[WX] "
```

**Output:**
```
[WX] 76#1pm,9,75,0.0#2pm,9,76,0.0#
```

### Example 4: Emoji Preamble

```yaml
preamble: "🌤️ "
```

**Output:**
```
🌤️ 76#1pm,9,75,0.0#2pm,9,76,0.0#
```

### Example 5: No Preamble (Default)

```yaml
preamble: ""
# or omit preamble entirely
```

**Output:**
```
#76#1pm,9,75,0.0#2pm,9,76,0.0#
```

The leading `#` is still present when no preamble is configured (backward compatible).

## Technical Details

### Code Change Location

**File:** `weather_formatter/formatter.py`

**Function:** `format_output()`

**Change:**
- Removed `_apply_preamble()` helper method
- Modified `format_output()` to use preamble OR entry separator at the start
- Preamble is now the first element if configured, otherwise entry separator is used

### Logic Flow

```python
# Old logic:
output_parts = [entry_separator]  # Always start with #
# ... build output ...
if preamble:
    output = preamble + output     # Prepend preamble → "WEATHER:#..."

# New logic:
if preamble:
    output_parts = [preamble]      # Start with preamble
else:
    output_parts = [entry_separator]  # Start with # (backward compatible)
# ... build output ...              # Result: "WEATHER:..." or "#..."
```

## Backward Compatibility

### No Preamble Configured
If you don't use a preamble, **no changes** to output:
```
#76#1pm,9,75,0.0#2pm,9,76,0.0#
```
This remains unchanged.

### Preamble Configured
If you use a preamble, the output **changes**:

**Before:** `WEATHER:#76#...`
**After:** `WEATHER:76#...`

To restore the old output, add `#` to your preamble: `"WEATHER:#"`

## Benefits

1. **More Control**: Users decide exactly what appears before the temperature
2. **Cleaner Prefixes**: Text preambles don't get unwanted separators
3. **Full Customization**: Any character sequence can be used as prefix
4. **Backward Compatible**: Default behavior (no preamble) unchanged

## Documentation Updates

The following documentation has been updated:

1. **README.md**
   - Updated "Using a Preamble" section with examples
   - Updated "Output Format" section with new structure
   - Updated example outputs

2. **examples/example_config.yaml**
   - Updated preamble examples
   - Added clarifying comments

3. **weather_formatter/formatter.py**
   - Updated docstrings
   - Added examples showing both behaviors

## Questions?

### Q: Do I need to change my config?
**A:** Only if you're using a preamble and want to keep the `#` after it. Add `#` to your preamble value.

### Q: Will this break my existing setup?
**A:** If you're not using a preamble, no. If you are using a preamble, the `#` after it will be removed.

### Q: Can I still have a separator after my preamble?
**A:** Yes! Just include it in the preamble itself. Example: `preamble: "WEATHER:#"`

### Q: What if I want a space after my preamble?
**A:** Include the space in your preamble. Example: `preamble: "WEATHER: "`
