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
  - `category` - Test category (e.g., `valid`, `invalid`)
  - `mode` - Parsing mode (e.g., `lax`, `strict`)

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
2. Parse the `in.maxi` file using your Maxi parser implementation
3. Compare your parser's output against `expected.json`
4. Validate specific assertions using `record_validations` and `object_validations`

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

- **valid** - Tests that should parse successfully
- **invalid** - Tests that should fail parsing (future)
- **edge-cases** - Tests for boundary conditions (future)

## Contributing

To add new test cases:

1. Create a new directory in `testdata/` with a unique 5-character ID
2. Add the required `test.json`, `in.maxi`, and `expected.json` files
3. Include any necessary external `.mxs` schema files
4. Ensure your test covers a specific aspect of the Maxi specification

## License

Released under the [MIT License](./LICENSE).

