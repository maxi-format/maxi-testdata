# Maxi Format Test Data

This repository contains test data for validating implementations of the [Maxi format](https://github.com/maxi-format/maxi) specification.

## Repository Structure

The test data is organized in the `testdata/` directory, where each subdirectory represents a specific test case with a unique ID.

```
testdata/
├── XXXX1/          # Test case 1
│   ├── test.json
│   ├── expected.json
│   └── in.maxi
├── XXXX2/          # Test case 2
│   ├── test.json
│   ├── expected.json
│   ├── in.maxi
│   └── schema_file.mxs
└── ...
```

## Test Case Structure

Each test case directory contains the following files:

### Required Files

- **`test.json`** - Test metadata including:
  - `id` - Unique test case identifier
  - `title` - Human-readable test name
  - `description` - Detailed description of what is being tested
  - `category` - Test category: `valid`, `error`, or `warning`
  - `mode` - Parsing mode: `lax` or `strict`
  - `tags` - Array of feature tags (see [Tag Reference](#tag-reference) below)

- **`in.maxi`** - The input Maxi format file to be parsed

- **`expected.json`** - Expected parser output for verification, containing:
  - `success` - Whether parsing should succeed (`true`/`false`)
  - `objects` - Object graph organized by type and ID
  - `records` - Sequential list of records as parsed
  - `record_validations` - Specific assertions for record data
  - `object_validations` - Specific assertions for object data (including reference following)

### Optional Files

- **`*.mxs`** - External schema files referenced by `@schema` directives in the input file

## Using the Test Data

### For Implementation Testing

1. Parse the `test.json` to understand the test requirements
2. Optionally filter tests by `tags` to skip features you haven't implemented yet
3. Parse the `in.maxi` file using your Maxi parser implementation
4. Compare your parser's output against `expected.json`
5. Validate specific assertions using `record_validations` and `object_validations`

### Filtering by Tags

The `tags` array in `test.json` lets you selectively run only the tests relevant to features your implementation supports. For example:

```js
// Skip tests that require fetching a remote schema URL
const tests = allTests.filter(t => !t.tags.includes('url-schema'));

// Only run basic inline-schema parse tests
const basic = allTests.filter(t =>
  t.tags.includes('inline-schema') &&
  !t.tags.includes('inheritance') &&
  !t.tags.includes('external-schema')
);

// Skip parser-dependent / optional behaviour tests
const normative = allTests.filter(t => !t.tags.includes('parser-dependent'));
```

### Validation Assertions

The `expected.json` includes two types of validations:

#### Record Validations
Test the sequential record output without following references:
```json
{
  "description": "Record 1 is User named Julie",
  "path": "#/records/0/value/name",
  "expected_value": "Julie"
}
```

#### Object Validations
Test the object graph, optionally following references:
```json
{
  "description": "Order 100 user name is Julie (follow ref)",
  "path": "#/objects/Order/100/user/name",
  "follow_references": true,
  "expected_value": "Julie"
}
```

## Test Categories

| Category | Meaning |
|----------|---------|
| `valid` | Input should parse successfully with no errors |
| `error` | Input should fail with a specific `error_code` in `expected.json` |
| `warning` | Input parses successfully (`success: true`) but emits one or more warnings |

---

## Tag Reference

Tags are grouped into four categories. Use them to skip tests for features your implementation hasn't yet built.

### Schema Source Tags

| Tag | Meaning |
|-----|---------|
| `inline-schema` | Type definitions are in the `in.maxi` file itself (before `###`) |
| `external-schema` | Schema loaded from a `.mxs` file via `@schema:path` |
| `url-schema` | Schema referenced via `@schema:https://…` URL |
| `no-schema` | No schema at all — data-only input, types unknown at parse time |
| `multi-schema` | Multiple `@schema` directives in the same file |
| `schema-override` | Later import overrides an earlier one (alias collision across imports) |

### Schema Feature Tags

| Tag | Meaning |
|-----|---------|
| `inheritance` | Type uses `<Parent>` single or multiple inheritance |
| `multi-inheritance` | Type inherits from two or more parent types |
| `alias-only` | Type definition uses bare alias form `U(…)` without `:TypeName` |
| `comments` | Schema or `.mxs` file contains `#` comment lines |
| `directives` | File uses at least one `@directive` (`@version`, `@mode`, `@schema`) |
| `enum` | Schema defines an `enum[…]` field |
| `constraints` | Schema defines field-level constraints (`>=N`, `<=N`, `=N`, pattern, `!`, `id`, …) |
| `defaults` | Schema defines default values on one or more fields |
| `required` | Schema marks at least one field as required `(!)` |
| `bytes` | Schema defines a `bytes` typed field |
| `annotations` | Schema uses type annotations (`@email`, `@url`, `@uuid`, `@date`, `@time`, `@timestamp`, `@hex`, `@base64`, `@datetime`) |
| `arrays` | Schema defines array-typed fields (`type[]`, `type[][]`, …) |
| `maps` | Schema defines map-typed fields (`map`, `map<K,V>`) |

### Data Feature Tags

| Tag | Meaning |
|-----|---------|
| `references` | Data records reference other records by ID |
| `inline-sub-object` | Data records contain inline sub-objects `(field\|(id\|name\|…))` |
| `multi-line-record` | Data record spans multiple lines inside `()` |
| `null-values` | Data records use `~` for explicit null |
| `empty-values` | Data records use `\|\|` for empty / omitted values |
| `quoted-strings` | Data records use `"…"` quoted string values |
| `escape-sequences` | Data records use `\n`, `\t`, `\r`, `\\`, `\"` inside quoted strings |
| `utf8` | Data records contain non-ASCII / UTF-8 characters |
| `bool-int` | Data records use `1`/`0` integer literals for bool fields |
| `array-data` | Data records contain array literals `[…]` |
| `map-data` | Data records contain map literals `{…}` |
| `object-array` | Array field whose elements are object records (inline or reference IDs) |

### Behavioural Tags

| Tag | Meaning |
|-----|---------|
| `strict` | File uses `@mode:strict` (explicit or inferred) |
| `lax` | File uses lax mode (default or explicit `@mode:lax`) |
| `forward-ref` | Data references an object whose record appears later in the file |
| `circular` | Schema has circular inheritance or circular import chain |
| `duplicate-id` | Data contains two records of the same type with the same identifier |
| `parser-dependent` | Test outcome is implementation-defined (optional behaviour per spec) |
| `error-code` | Test expects a specific `error_code` in `expected.json` |
| `warning` | Test expects a warning but `success: true` |

---

## Contributing

To add new test cases:

1. Create a new directory in `testdata/` with a unique 5-character ID
2. Add the required `test.json`, `in.maxi`, and `expected.json` files
3. Include any necessary external `.mxs` schema files
4. Add a `tags` array to `test.json` using the tags from the [Tag Reference](#tag-reference) above
5. Ensure your test covers a specific aspect of the Maxi specification

## License

Released under the [MIT License](./LICENSE).
