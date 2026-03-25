# Implementation notes issue #678

## API

* The `test` command shall have a `--check` parameter.
Its type is `List[CheckType]`.
This allows for a passing the value multiple times, e.g.
```shell
datacontract test mycontract.yaml --server production --check=schema --check=quality
```

In the example above, the argument `check` will have value `[CheckType.schema, CheckType.quality]`

## Tests

There seems to be a naming convention for tests...

```yaml
- schema:
    fail:
      when wrong column type:
        field_three: timestamptz
    name: my_table__field_three__field_type
- my_table__quality_sql_0 < 3600:
    my_table__quality_sql_0 query: "SELECT MAX(duration) AS max_duration\nFROM (\n\
      \  SELECT EXTRACT(EPOCH FROM (field_three - LAG(field_three) OVER (ORDER BY\
      \ field_three))) AS duration\n  FROM my_table\n) subquery;\n"
    name: my_table__quality_sql_0
```

But maybe this can be influenced by `quality.id`, thereby becoming unreliable for filtering.

Also, the run result has a list of checks:

```json
{
  "id": "a055ace7-ee59-4154-becf-84a59a8c22dd",
  "key": "my_table__field_one__field_required",
  "category": "schema",
  "type": "field_required",
  "name": "Check that field field_one has no missing values",
  "model": "my_table",
  "field": "field_one",
  "engine": "soda",
  "language": "sodacl",
  "implementation": "checks for \"my_table\":\n- missing_count(\"field_one\") = 0:\n    name: my_table__field_one__field_required\n",
  "result": "passed",
  "reason": "",
  "details": null,
  "diagnostics": {
    "blocks": [],
    "value": 0,
    "fail": {
      "greaterThan": 0.0,
      "lessThan": 0.0
    }
  }
}
```

The `$.type` property might be suitable for filtering.
This comes from `data_contract_checks.py` (L267).
It seems this file contains a large number of types, consistently assigned to the `check_types` variable.
It makes sense to move these types into en enum, that we can filter on.
The types can then be added to a `check_type_group`, which is used as argument.

## Testing

### Running tests

```shell
pytest tests/test_test_quality.py -s --log-cli-level=DEBUG  
```

### Tests to create

* Given contract with full spec quality, assert that groups are executed based on params
* Assert default all tests are executed

## UV

```shell
uv sync --all-groups --all-extras   
```