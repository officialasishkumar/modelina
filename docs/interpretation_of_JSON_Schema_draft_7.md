# Interpretation of JSON Schema draft 7 to CommonModel

The library transforms JSON Schema from data validation rules to data definitions (`CommonModel`(s)). 

The algorithm tries to get to a model whose data can be validated against the JSON schema document. 

We only provide the underlying structure of the schema file for the model formats such as `maxItems`, `uniqueItems`, `multipleOf`, etc, are not transformed.

## Interpreter 
The main functionality is located in the `Interpreter` class. This class ensures to recursively create (or retrieve from a cache) a `CommonModel` representation of a Schema. We have tried to keep the functionality split out into separate functions to reduce complexity and ensure it is easy to maintain. This main function also ensures to split any created models into separate ones if needed.

The order of transformation:
- [type](#determining-the-type-for-the-model)
- `required` are interpreted as is.
- `properties` are interpreted as is, where duplicate `properties` for the model are merged.
- [allOf](#allOf-sub-schemas)
- `enum` are interpreted as is, where each `enums`.
- `const` interpretation overwrite already interpreted `enums`.
- `items` are interpreted as is, where more then 1 item are merged.
- `additionalProperties` are interpreted as is, where duplicate additionalProperties for the model are merged together. If the schema does not define additionalProperties it defaults to `true` schema.
- `patternProperties` are interpreted as is, where duplicate patterns for the model are merged together.
- [oneOf/anyOf/then/else](#Processing-sub-schemas)
- [not](#interpreting-not-schemas)

## Interpreting not schemas
`not` schemas infer the form for which the model should not take by recursively interpret the `not` schema, and remove certain model properties when encountered.

Currently the following `not` model properties are interpreted:
- `type`
- `enum`

**Restrictions** 
- You cannot use nested `not` schemas to infer new model properties, it can only be used to re-allow them.
- boolean `not` schemas are not applied.

## allOf sub schemas
`allOf` are a bit different then the other [combination keywords](#Processing-sub-schemas) since it can imply inheritance. 

So dependant on whether the simplify option `allowInheritance` is true or false we interpret it as inheritance or simply merge the models together.

## Determining the type for the model
To determine the types for the model we use the following interpretation (and in that order):
- `true` schema infers all model types (`object`, `string`, `number`, `array`, `boolean`, `null`, `integer`).
- Usage of `type` infers the initial model type.
- Usage of `properties` infers `object` model type.
- Usage of `const` infers the constant value as type if schema does not have `type` specified.
- Usage of `enum` infers the enum values as type if schema does not have `type` specified.

## Processing sub schemas
The following JSON Schema keywords are merged with the already interpreted model:
- `oneOf`
- `anyOf`
- `then`
- `else`