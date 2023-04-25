# Filter Expressions

Filter expressions in STTP are used to select desired signals for subscription or to reduce available meta-data down to a desired subset. Filtering syntax is similar to [Structured Query Language](https://en.wikipedia.org/wiki/SQL) (SQL), but does not implement the full SQL language.

Filter expressions operate against in-memory [data set](../data-sets), not a backend database. The filtering syntax used in conjunction with a data set is designed for read-only operations and exposes no update functionality. Because of this, filter operations are not subject to SQL injection attacks or other security concerns typically associated with SQL implementations.

STTP data publishers need to define a data set consisting of a collection of data tables representing the [primary meta-data](#primary-meta-data-table-definitions) from locally defined configurations that contain information about the time-series style data to be published. At a minimum this meta-data should define a [Guid](https://en.wikipedia.org/wiki/Universally_unique_identifier) based identifier for each [measurement](#measurementdetail) to be published as well as an associated source, i.e., a [device](#devicedetail), that produces the measurement.

> :information_source: The STTP data publisher API defines functions to help create needed meta-data, see [samples](https://github.com/sttp/cppapi/tree/master/src/samples) and [specific example](https://github.com/sttp/cppapi/blob/master/src/samples/InstancePublish/PublisherHandler.cpp#L72)

## Filtering Syntax
```sql
FILTER [TOP n] <TableName> WHERE <Expression> [ORDER BY <SortField> [ASC|DESC]]
```

Filter expressions in STTP are parsed using [ANTLR](https://www.antlr.org/). For complete syntax description, see full ANTLR grammar: **[FilterExpressionSyntax.g4](https://github.com/sttp/filter-expressions/blob/main/FilterExpressionSyntax.g4)**

### Available Options and Clauses

| Keyword | Example | Description | Required?|
|---------|---------|-------------|:----------:|
| `FILTER` | See [examples](#examples) below | Keyword that signifies a _filter_ expression follows* | Yes|
| `TOP n` | `TOP 100` | Selects only the first `n` number of items | No|
| `WHERE <Expression>` | `WHERE SignalType='FREQ'` | Criteria based expression, in SQL syntax, used to filter rows | Yes |
| `ORDER BY <ColumnName>` | `ORDER BY SignalType` | Expression specifying column names and sort directions | No |

> :information_source: The keyword _FILTER_ is used instead of the standard SQL _SELECT_ keyword to reinforce the notion that the expression that follows is special purposed and not standard SQL.

### Direct Signal Identification

Filtering syntax also supports the direct specification of desired signals as semi-colon separated measurement references in a variety of forms, e.g., **measurement key identifiers**: _PPA:4; PPA:2_ - formatted as `{instance}:{id}`, unique **Guid-based signal identifiers**: _538A47B0-F10B-4143-9A0A-0DBC4FFEF1E8; '06d039f6-e5e9-4e37-85fc-52a125c67a06'; {E4BBFE6A-35BD-4E5B-92C9-11FF913E7877}_ optionally surrounded by single quotes or braces, or **point tag name identifiers**: _"GPA_TESTDEVICE:FREQ"; "GPA_TESTDEVICE:FLAG"_ where each point tag name is in double quotes.

### Examples

Example filter expression to select measurements with the company of `GPA` and type of Frequency `(FREQ)` or Voltage Magnitude `(VPHM)`:
```sql
    FILTER ActiveMeasurements WHERE Company='GPA' AND SignalType IN ('FREQ', 'VPHM') ORDER BY Device DESC
```

Example filter expression to select first 20 measurements of type Statistic `(STAT)`:
```sql
    FILTER TOP 20 ActiveMeasurements WHERE SignalType = 'STAT'
```

Example filter to only select Current Angle `(IPHA)` and Voltage Angle `(VPHA)` for Positive Sequence `(+)` measurements.
```sql
    FILTER ActiveMeasurements WHERE SignalType LILE '%PHA' AND Phase='+' ORDER BY PhasorID
```

Example filter combining both filter expressions and directly specified tags:

```sql
PPA:15; STAT:20; PPA:8; {eecbda2f-fe76-4504-b031-7f5518c7046c};
FILTER ActiveMeasurements WHERE SignalType IN ('IPHA', 'VPHA'); 9d0423c0-2349-4a38-85d5-b6e81735eb48;
FILTER TOP 3 ActiveMeasurements WHERE SignalType = 'FREQ' ORDER BY Device; "GPA_TESTDEVICE:FREQ"
```

### Filter Expression Operators

Any operators that consist of letters, e.g., `OR`, are not case sensitive, so `OR`, `or` and `Or` are all equivalent.

#### Unary Operators

| Type | Symbol |
| :--: | :----: |
| Negative | `-` |
| Positive | `+` |
| Not | `NOT` or `!` or `~` |

#### Comparison Operators

| Type | Symbol |
| :--: | :----: |
| Less Than | `<` |
| Less Than or Equal | `<=` |
| Greater Than | `>` |
| Greater Than or Equal | `>=` |
| Equal | `=` or `==` or [`===`](#case-sensitive-comparison-operators) |
| Not Equal | `!=` or `<>` or [`!==`](#case-sensitive-comparison-operators) |

#### Logical Operators

| Type | Symbol |
| :--: | :----: |
| And | `AND` or `&&` |
| Or | `OR` or `||` <!-- Do not modify: the `||` renders improperly on Github, but works fine on jekyll. `\|\|` works but is unescaped in jekyll --> |

#### Bitwise Operators

| Type | Symbol |
| :--: | :----: |
| Bit Shift Left | `<<` |
| Bit Shift Right | `>>` |
| Bitwise And | `&` |
| Bitwise Or | `|` <!-- Do not modify: the `|` renders improperly on Github, but works fine on jekyll. `\|` works but is unescaped in jekyll --> |
| Exclusive Or | `XOR` or `^` |

#### Math Operators

| Type | Symbol |
| :--: | :----: |
| Multiply | `*` |
| Divide | `/` |
| Add | `+` |
| Subtract | `-` |
| Modulus | `%` |

#### Other Operators

| Symbol | Syntax |
| :----: | :----: |
| `LIKE` | `<ColumnName> LIKE 'pattern'`
|  `IN`  | `<ColumnName> IN (expression1, ..., expression_n)` |

> The _pattern_ for the `LIKE` operator uses wildcards identified as a percent sign (`%`), or an asterisk (`*`), allowed at the beginning or the end of the pattern, which represents zero, one, or multiple characters in the source column value. Wildcards in the middle of the string are not supported.

### Case Sensitive String Comparisons

Unless otherwise specified, comparison of string values in filter expressions is not case sensitive. To specify a case sensitive comparison, use one of the following expression options:

#### Case Sensitive `LIKE` Expression

```
FILTER <TableName> WHERE <ColumnName> [NOT] LIKE [===|BINARY] 'pattern'
```
_Example:_
```
FILTER ActiveMeasurements WHERE Device LIKE BINARY 'SHELBY%'
```

#### Case Sensitive `IN` Expression

```
FILTER <TableName> WHERE expression [NOT] <ColumnName> IN [===|BINARY] (expression1, ..., expression_n )
```
_Example:_
```
FILTER ActiveMeasurements WHERE NOT SignalType IN ===('IPHM', 'VPHM')
```

#### Case Sensitive `ORDER BY` Expression

```
FILTER <TableName> WHERE expression ORDER BY [===|BINARY] <ColumnName> [ASC|DESC]
```
_Example:_
```
FILTER TOP 5 ActiveMeasurements ORDER BY Device, === PointTag DESC
```

#### Case Sensitive Comparison Operators

When expressions are strings, or evaluated as strings, the following operators perform a case sensitive compare:
```
expression1 === expression2   // Case sensitive equals comparison
expression1 !== expression2   // Case sensitive not equals comparison
```
_Example:_
```
FILTER ActiveMeasurements WHERE Device === 'SHELBY'
```

## Filter Expression Functions

Function names are not case sensitive, so `ABS`, `abs` and `Abs` are all equivalent.

| Function | Arguments | Description |
| :------: | :-------: | ----------- |
| `ABS` | `expression` | Returns the absolute value of the specified numeric `expression`. |
| `CEILING` | `expression` | Returns the smallest _integer_ that is greater than, or equal to, the specified numeric `expression`. |
| `COALESCE` | `expression1`, ..., `expression_n` | Returns the first _non-null_ value in `expression` list. |
| `CONVERT` | `expression`, `type` | Returns `expression` converted to the specified `type`. `type` is one of _boolean_ (or _bool_), _int32_, _uint_, _int64_ (or _int_), _decimal_, _single_ (or _float_), _double_, _string_, _UUID_ (or _GUID_),or _datetime_. `type` is not case sensitive. |
| `CONTAINS` | `source`, `test`, [`ignorecase`] | Returns _boolean_ flag that determines if `source` _string_ contains `test` _string_. `ignorecase` is an optional _boolean_ flag, defaults to _false_, to determine if string comparison is case sensitive. |
| `DATEADD` | `source`, `value`, `interval` | Returns _dateTime_ of _integer_ `value` at specified `interval` added to `source` _dateTime_ value. `interval` is one of _Year_, _Month_, _DayOfYear_, _Day_, _Week_, _WeekDay_, _Hour_, _Minute_, _Second_, or _Millisecond_. `interval` is not case sensitive. |
| `DATEDIFF` | `left`, `right`, `interval` | Returns the _integer_ difference between `left` and `right` _dateTime_ value at specified `interval`. `interval` is one of _Year_, _Month_, _DayOfYear_, _Day_, _Week_, _WeekDay_, _Hour_, _Minute_, _Second_, or _Millisecond_. `interval` is not case sensitive. |
| `DATEPART` | `source`, `interval` | Returns specified _integer_ `interval` of `source` _dateTime_ value. `interval` is one of _Year_, _Month_, _DayOfYear_, _Day_, _Week_, _WeekDay_, _Hour_, _Minute_, _Second_, or _Millisecond_. `interval` is not case sensitive. |
| `ENDSWITH` | `source`, `test`, [`ignorecase`] | Returns _boolean_ flag that determines if `source` _string_ ends with `test` _string_. `ignorecase` is an optional _boolean_ flag, defaults to _false_, to determine if string comparison is case sensitive. |
| `FLOOR` | `expression` | Returns the largest _integer_ value that is smaller than, or equal to, the specified numeric `expression`. |
| `IIF` | `expression`, `leftValue`, `rightValue` | Returns `leftValue` if result of `expression` is _true_, else returns `rightValue`. |
| `INDEXOF` | `source`, `test`, [`ignorecase`] | Returns zero-based _integer_ index of first occurrence of `test` _string_ in `source` _string_, or _-1_ if not found. `ignorecase` is an optional _boolean_ flag, defaults to _false_, to determine if string comparison is case sensitive. |
| `ISDATE` | `expression` | Returns _boolean_ flag that determines if `expression` is a _dateTime_ or can be parsed as one. |
| `ISINTEGER` | `expression` | Returns _boolean_ flag that determines if `expression` is an _integer_ value or can be parsed as one. |
| `ISUUID` | `expression` | Returns _boolean_ flag that determines if `expression` is a _UUID_ value or can be parsed as one. |
| `ISNULL` | `expression` | Returns _boolean_ flag that determines if `expression` is _null_. |
| `ISNUMERIC` | `expression` | Returns _boolean_ flag that determines if `expression` is a numeric value or can be parsed as one. |
| `LASTINDEXOF` | `source`, `test`, [`ignorecase`] | Returns zero-based _integer_ index of last occurrence of `test` _string_ in `source` _string_, or _-1_ if not found. `ignorecase` is an optional _boolean_ flag, defaults to _false_, to determine if string comparison is case sensitive. |
| `LEN` | `expression` | Returns _integer_ length of `expression` interpreted as a _string_. |
| `LOWER` | `expression` | Returns lower-case representation of `expression` interpreted as a _string_. |
| `MAXOF` | `expression1`, ..., `expression_n` | Returns value in `expression` list with maximum value. |
| `MINOF` | `expression1`, ..., `expression_n` | Returns value in `expression` list with minimum value. |
| `NOW` | | Returns _dateTime_ value representing the current local system time. |
| `NTHINDEXOF` | `source`, `test`, `index`, [`ignorecase`] | Returns zero-based _integer_ index of the Nth, represented by _integer_ `index` value, occurrence of `test` _string_ in `source` _string_, or _-1_ if not found. `ignorecase` is an optional _boolean_ flag, defaults to _false_, to determine if string comparison is case sensitive. |
| `POWER` | `expression`, `exponent` | Returns the value of specified numeric `expression` raised to the power of specified numeric `exponent`. |
| `REGEXMATCH` | `regex`, `test` | Returns _boolean_ flag that determines if `test`, interpreted as a _string_, is a match for specified `regex` string-based regular expression. |
| `REGEXVAL` | `regex`, `test` | Returns _string_ value from `test`, interpreted as a _string_, that is matched by specified `regex` string-based regular expression. |
| `REPLACE` | `source`, `test`, `replace`, [`ignorecase`] | Returns _string_ where all instances of `test` found in `source` are replaced with `replace` value - all parameters interpreted as _strings_. `ignorecase` is an optional _boolean_ flag, defaults to _false_, to determine if string comparison is case sensitive. |
| `REVERSE` | `expression` | Returns _string_ where all characters in `expression`, interpreted as a _string_, are reversed. |
| `ROUND` | `expression` | Returns the nearest _integer_ value to the specified numeric `expression`. |
| `SPLIT` | `source`, `delimiter`, `index`, [`ignorecase`] | Returns zero-based Nth, represented by `index`, _string_ value in `source` _string_ split by `delimiter` _string_, or _null_ if out of range. `ignorecase` is an optional _boolean_ flag, defaults to _false_, to determine if string comparison is case sensitive. |
| `SQRT` | `expression` | Returns the square root of the specified numeric `expression`. |
| `STARTSWITH` | `source`, `test`, [`ignorecase`] | Returns _boolean_ flag that determines if `source` _string_ starts with `test` _string_. `ignorecase` is an optional _boolean_ flag, defaults to _false_, to determine if string comparison is case sensitive. |
| `STRCOUNT` | `source`, `test`, [`ignorecase`] | Returns _integer_ count of occurrences of `test` _string_ in `source` _string_. `ignorecase` is an optional _boolean_ flag, defaults to _false_, to determine if string comparison is case sensitive. |
| `STRCMP` | `left`, `right`, [`ignorecase`] | Returns _integer_ _-1_ if `left` _string_ is less-than `right` _string_, _1_ if `left` _string_ is greater-than `right` _string_, or _0_ if `left` _string_ equals `right` _string_. `ignorecase` is an optional _boolean_ flag, defaults to _false_, to determine if string comparison is case sensitive. |
| `SUBSTR` | `source`, `index`, [`length`] | Returns _string_ from portion of `source`, interpreted as a _string_, starting at _integer_ `index`. If `length` is specified, this will be the maximum number of characters returned; otherwise, remaining characters in `source` _string_ will be returned. |
| `TRIM` | `expression` | Returns _string_ with white-space removed from the beginning and end of `expression` interpreted as a _string_. |
| `TRIMLEFT` | `expression` | Returns _string_ with white-space removed from the beginning of `expression` interpreted as a _string_. |
| `TRIMRIGHT` | `expression` | Returns _string_ with white-space removed the end of `expression` interpreted as a _string_. |
| `UPPER` | `expression` | Returns upper-case representation of `expression` interpreted as a _string_. |
| `UTCNOW` | | Returns _dateTime_ value representing the current UTC system time. |

## Signal Selection Meta-data Table Definitions

Data publishers can define multiple tables that represent sets of measurements available for filtering desired signals, e.g., `AllMeasurements` or `LocalMeasurements`. At a minimum a signal selection table must define a `SignalD` field of type `Guid` - all other fields are considered optional. However, without a point tag name or description the measurement may be of little use unless other meta-data is exchanged out-of-band with STTP.

Signal selection tables should represent a simple flattened "_view_" of available meta-data with as many fields as needed to be useful for measurement selection operations. See usage of `ActiveMeasurements` in [examples](#examples).

### _ActiveMeasurements_

The `ActiveMeasurements` table is always expected to be defined. This table represents all measurements considered _active_ and _available_ for subscription. If a data publisher is controlling access to measurements on a per-subscriber basis, this table should only include the measurements the subscriber is allowed to request for subscription.

Typically the data in the `ActiveMeasurements` table is derived from the conflation of information already defined in other available meta-data condensed to a single table to make filter expressions more productive.

Common fields for the `ActiveMeasurements` table are defined below. Note that some of the fields are specific to the electric power industry and may not be applicable for other industry implementations and consequently unavailable.

> :information_source: The STTP data publisher API will automatically generate the `ActiveMeasurements` table when [primary meta-data tables](#primary-meta-data-table-definitions) are defined, see the [DefineMetadata](https://github.com/sttp/cppapi/blob/master/src/lib/transport/DataPublisher.h#L155) function.

| Column Name | Data Type | Description |
| ----------: | :-------: | :---------- |
| ID | string | A measurement key identifier formatted as `{instance}:{id}` |
| SignalID | Guid | Unique identifier for the measured value |
| PointTag | string | Unique alpha-numeric identifier for the measured value |
| AlternateTag | string | Secondary alpha-numeric identifier for the measured value |
| SignalReference | string | Alpha-numeric reference to original signal source, e.g., location in source protocol |
| Device | string | Alpha-numeric device acronym that is the source of the measurement |
| FramesPerSecond | int | Expected data rate, in received samples per second, of measurement |
| Protocol | string | Source protocol that generated measurement |
| SignalType | string | [Signal type acronym](https://github.com/sttp/cppapi/blob/master/src/lib/transport/TransportTypes.cpp#L210) of measurement |
| EngineeringUnits | string | Engineering units of measurement |
| PhasorID | int | ID of associated [phasor meta-data](#phasordetail) record |
| PhasorType | string | When measurement is a phasor, type of phasor: voltage (`V`) or current (`I`) |
| Phase | string | When measurement is a phasor, phase e.g.: (`A`), (`B`), (`C`), (`+`), (`-`), etc. |
| Adder | double | Recommended additive linear adjustment of value to be applied |
| Multiplier | double | Recommended multiplicative linear adjustment of value to be applied |
| Company | string | Acronym of company that is publishing the measurement |
| Longitude | decimal | Longitude of device location publishing the measurement |
| Latitude | decimal | Latitude of the device location publishing the measurement |
| Description | string | Description of the measurement |
| UpdatedOn | dateTime | Timestamp of last update of measurement meta-data |

## Primary Meta-data Table Definitions

STTP meta-data is designed around the notion of a [data set](https://github.com/sttp/cppapi/tree/master/src/lib/data). Meta-data represented by a data set allows for rich and extensible information description.

Outside the expected `ActiveMeasurements` signal selection meta-data table definition, no other meta-data tables are required to be defined. However, to make data exchange useful for industry specific STTP implementations, a common set of meta-data should be defined.

The STTP data publisher API currently defines three primary data tables to define enough useful meta-data to allow a measurement data subscription to be converted into another protocol, e.g., [IEEE C37.118](https://standards.ieee.org/standard/C37_118_1-2011.html). When these tables are defined, the data publisher API will auto-generate the `ActiveMeasurements` table from the provided data.

### _DeviceDetail_

This meta-data table contains details about the devices that are the sources of available measurements. By convention, [measurements](#measurementdetail) that are not associated with a device are not sent in meta-data exchanges.

| Column Name | Data Type | Description |
| ----------: | :-------: | :---------- |
| UniqueID | Guid |  Unique identifier for the device |
| OriginalSource | string | If device was proxied through another protocol, original source |
| IsConcentrator | boolean | Flag that determines if device is a container for other devices |
| Acronym | string | Alpha-numeric device acronym |
| Name | string | Free form device name |
| AccessID | int | ID code associated with device, if any |
| ParentAcronym | string | If device is a child of another device, acronym of parent device |
| FramesPerSecond | int | Expected data rate, in received samples per second, for device measurements |
| CompanyAcronym | string | Company that owns device |
| VendorAcronym | string | Vendor that manufactures device |
| VendorDeviceName | string | Vendor device name and/or model information |
| Longitude | decimal | Longitude of device location |
| Latitude | decimal | Latitude of the device location |
| InterconnectionName | string | Eastern, Western, etc. |
| ContactList | string | Names / e-mail addresses of parties responsible for device |
| Enabled | boolean | Flag that determines if device is currently enabled |
| UpdatedOn | dateTime |Timestamp of last update of device meta-data |

### _MeasurementDetail_

This meta-data table contains details about the measurements available for subscription.

| Column Name | Data Type | Description |
| ----------: | :-------: | :---------- |
| DeviceAcronym | string |Alpha-numeric device acronym that is the source of the measurement |
| ID | string | A measurement key identifier formatted as `{instance}:{id}` |
| SignalID | Guid | Unique identifier for the measured value |
| PointTag | string | Unique alpha-numeric identifier for the measured value |
| SignalReference | string | Alpha-numeric reference to original signal source, e.g., location in source protocol |
| SignalAcronym | string | Type of signal, e.g., `FREQ` for frequency |
| PhasorSourceIndex | int | Index of phasor source if measurement type is a phasor |
| Description | string | Description of the measurement |
| Enabled | boolean | Flag that determines if measurement is currently enabled |
| UpdatedOn | dateTime | Timestamp of last update of measurement meta-data |

### _PhasorDetail_

This meta-data table, specific to data exchanges containing electrical measurements with [phasor](https://en.wikipedia.org/wiki/Phasor) values, contains details about the phasors whose vector magnitude and angle component measurements are available for subscription.

| Column Name | Data Type | Description |
| ----------: | :-------: | :---------- |
| ID | int | Numeric auto-incrementing identifier |
| DeviceAcronym | string |Alpha-numeric device acronym that is the source of the phasor |
| Label | string | Free form phasor label |
| Type | string | Type of phasor, i.e.: voltage (`V`) or current (`I`) |
| Phase | string | Phase of phasor, e.g.: (`A`), (`B`), (`C`), (`+`), (`-`), etc. |
| DestinationPhasorID | int | Associated phasor, e.g., typical voltage for current |
| SourceIndex | int | Index of phasor as defined in source protocol |
| UpdatedOn | dateTime | Timestamp of last update of DestinationPhasorID meta-data |
