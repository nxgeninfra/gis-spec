# Contributing to GIS

Thank you for your interest in the GPU Instance Specification. Contributions make the spec better for everyone.

## Ways to Contribute

### Add a Provider Example

The most impactful contribution is adding a GIS document for a provider not yet covered.

1. Create a file in `examples/` named `{provider}-{gpu}.json`
2. Follow the schema at `schema/v1.json`
3. Include a `source_url` pointing to the provider's pricing page
4. Compute the `normalized` values using the formulas in [NORMALIZATION.md](./NORMALIZATION.md)
5. Set `verified` to `true` only if you manually confirmed the data
6. Submit a PR

**Naming convention:** `{provider-kebab-case}-{gpu-model}.json`
- `lambda-h100.json` ✅
- `Lambda_H100.json` ❌

### Report Incorrect Data

If pricing or specs are wrong in an existing example:

1. Open an issue with the correct values
2. Include the source URL where you found the updated data
3. Or submit a PR with the fix

### Spec Feedback

For changes to the spec itself (new fields, type changes, naming):

1. Open an issue describing the change and rationale
2. Include which providers would benefit
3. Reference the [design principles](./README.md#design-principles)

Breaking changes require a major version bump. New optional fields are minor.

## Validation

Before submitting, validate your GIS document:

```bash
npx ajv validate -s schema/v1.json -d examples/your-file.json
```

## Code of Conduct

See the [org-level Code of Conduct](https://github.com/nxgeninfra/.github/blob/main/CODE_OF_CONDUCT.md).

## Questions?

- Email: [spec@computespec.dev](mailto:spec@computespec.dev)
- Issues: [github.com/nxgeninfra/gis-spec/issues](https://github.com/nxgeninfra/gis-spec/issues)

---

*All contributions to this specification are licensed under CC BY 4.0.*
