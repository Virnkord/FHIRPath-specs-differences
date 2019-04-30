FHIRPath (3rd Normative Ballot)
FHIRPath is a path based navigation and extraction language, somewhat like XPath. Operations are expressed in terms of the logical content of hierarchical data models, and support traversal, selection and filtering of data. Its design was influenced by the needs for path navigation, selection and formulation of invariants in both HL7 Fast Healthcare Interoperability Resources (FHIR) and HL7 Clinical Quality Language (CQL).

Looking for implementations? See FHIRPath Implementations on the HL7 wiki

Version: 1.3.0 Public Domain (Creative Commons 0)

Note: The following sections of this specification have not received significant implementation experience and are marked for Standard for Trial Use (STU):

Aggregates

Types - Reflection

Functions - Math

In addition, the appendices are included as additional documentation and are informative content.

Table of Contents
1. Background
1.1. Requirements
1.2. Features
1.3. Usage
1.4. Conventions
2. Navigation model
3. Path selection
3.1. Collections
3.2. Paths and polymorphic items
4. Expressions
4.1. Literals
4.2. Operators
4.3. Function Invocations
4.4. Null and empty
4.5. Singleton Evaluation of Collections
5. Functions
5.1. Existence
5.2. Filtering and projection
5.3. Subsetting
5.4. Combining
5.5. Conversion
5.6. String Manipulation
5.7. Math
5.8. Tree navigation
5.9. Utility functions
6. Operations
6.1. Equality
6.2. Comparison
6.3. Types
6.4. Collections
6.5. Boolean logic
6.6. Math
6.7. Date/Time Arithmetic
6.8. Operator precedence
7. Aggregates
7.1. aggregate(aggregator : expression [, init : value]) : value
8. Lexical Elements
8.1. Whitespace
8.2. Comments
8.3. Literals
8.4. Symbols
8.5. Keywords
8.6. Identifiers
8.7. Case-Sensitivity
9. Environment variables
10. Types and Reflection
10.1. Models
10.2. Reflection
11. Type safety and strict evaluation
12. Formal Specifications
12.1. Formal Syntax
12.2. Model Information
12.3. URI and Media Types
Appendix A: Use of FHIRPath on HL7 Version 2 messages
Appendix B: FHIRPath Tooling and Implementation
Appendix C: References
1. Background
In Information Systems in general, and Healthcare Information Systems in particular, the need for formal representation of logic is both pervasive and critical. From low-level technical specifications, through intermediate logical architectures, up to the high-level conceptual descriptions of requirements and behavior, the ability to formally represent knowledge in terms of expressions and information models is essential to the specification and implementation of these systems.

1.1. Requirements
Of particular importance is the ability to easily and precisely express conditions of basic logic, such as those found in requirements constraints (e.g. Patients must have a name), decision support (e.g. if the patient has diabetes and has not had a recent comprehensive foot exam), cohort definitions (e.g. All male patients aged 60-75), protocol descriptions (e.g. if the specimen has tested positive for the presence of sodium), and numerous other environments.

Precisely because the need for such expressions is so pervasive, there is no shortage of existing languages for representing them. However, these languages tend to be tightly coupled to the data structures, and even the information models on which they operate, XPath being a typical example. To ensure that the knowledge captured by the representation of these expressions can survive technological drift, a representation that can be used independent of any underlying physical implementation is required.

Languages meeting these additional requirements also exist, such as OCL, Java, JavaScript, C#, and others. However, these languages are both tightly coupled to the platforms in which they operate, and, because they are general-purpose development languages, come with much heavier tooling and technology dependencies than is warranted or desirable. Even constraining one of these grammars would be insufficient, resulting in the need to extend, defeating the purpose of basing it on an existing language in the first place.

Given these constraints, and the lack of a specific language that meets all of these requirements, there is a need for a simple, lightweight, platform- and structure-independent graph traversal language. FHIRPath meets these requirements, and can be used within various environments to provide for simple but effective formal representation of expressions.

1.2. Features
Graph-traversal: FHIRPath is a graph-traversal language; authors can clearly and concisely express graph traversal on hierarchical information models (e.g. HL7 V3, FHIR, vMR, CIMI, and QDM).

Fluent: FHIRPath has a syntax based on the Fluent Interface pattern

Collection-centric: FHIRPath deals with all values as collections, allowing it to easily deal with information models with repeating elements.

Platform-independent: FHIRPath is a conceptual and logical specification that can be implemented in any platform.

Model-independent: FHIRPath deals with data as an abstract model, allowing it to be used with any information model.

1.3. Usage
In Fast Healthcare Interoperability Resources (FHIR), FHIRPath is used within the specification to provide formal definitions for conditions such as validation invariants, search parameter paths, etc. Within Clinical Quality Language (CQL), FHIRPath is used to simplify graph-traversal for hierarchical information models.

In both FHIR and CQL, the model independence of FHIRPath means that expressions can be written that deal with the contents of the resources and data types as described in the Logical views, or the UML diagrams, rather than against the physical representation of those resources. JSON and XML specific features are not visible to the FHIRPath language (such as comments and the split representation of primitives (i.e. value[x])).

The expressions can in theory be converted to equivalent expressions in XPath, OCL, or another similarly expressive language.

FHIRPath can be used against many other graphs as well. For example, Use of FHIRPath on HL7 Version 2 messages describes how FHIRPath is used in HL7 V2.

1.4. Conventions
Throughout this documentation, monospace font is used to delineate expressions of FHIRPath.

Optional parameters to functions are enclosed in square brackets in the definition of a function. Note that the brackets are only used to indicate optionality in the signature, they are not part of the actual syntax of FHIRPath.

All operations and functions return a collection, but if the operation or function will always produce a collection containing a single item of a predefined type, the description of the operation or function will specify its output type explicitly, instead of just stating collection, e.g. all(…​) : Boolean

Formatting strings for Date, Time, and DateTime values are described using the following markers:

YYYY - A full four digit year, padded with leading zeroes if necessary

MM - A full two digit month value, padded with leading zeroes if necessary

DD - A full two digit day value, padded with leading zeroes if necessary

HH - A full two digit hour value (00..24), padded with leading zeroes if necessary

mm - A full two digit minute value (00..59), padded with leading zeroes if necessary

ss - A full two digit second value (00..59), padded with leading zeroes if necessary

fff - A fractional millisecond value (0..999)

2. Navigation model
FHIRPath navigates and selects nodes from a tree that abstracts away and is independent of the actual underlying implementation of the source against which the FHIRPath query is run. This way, FHIRPath can be used on in-memory Java POJOs, Xml data or any other physical representation, so long as that representation can be viewed as classes that have properties. In somewhat more formal terms, FHIRPath operates on a directed acyclic graph of classes as defined by a MOF-equivalent [MOF] type system.

Data are represented as a tree of labelled nodes, where each node may optionally carry a primitive value and have child nodes. Nodes need not have a unique label, and leaf nodes must carry a primitive value. For example, a (partial) representation of a FHIR Patient resource in this model looks like this:

Tree representation of a Patient

The diagram shows a tree with a repeating name node, which represents repeating members of the FHIR object model. Leaf nodes such as use and family carry a (string) value. It is also possible for internal nodes to carry a value, as is the case for the node labelled active: this allows the tree to represent FHIR "primitives", which may still have child extension data.

FHIRPath expressions are then evaluated with respect to a specific instance, such as the Patient one described above. This instance is referred to as the context (also called the root) and paths within the expression are evaluated in terms of this instance.

3. Path selection
FHIRPath allows navigation through the tree by composing a path of concatenated labels, e.g.

name.given
This would result in a collection of nodes, one with the value 'Wouter' and one with the value 'Gert'. In fact, each step in such a path results in a collection of nodes by selecting nodes with the given label from the step before it. The input collection at the beginning of the evaluation contained all elements from Patient, and the path name selected just those named name. Since the name element repeats, the next step given along the path, will contain all nodes labeled given from all nodes name in the preceding step.

The path may start with the type of the root node (which otherwise does not have a name), but this is optional. To illustrate this point, the path name.given above can be evaluated as an expression on a set of data of any type. However the expression may be prefixed with the name of the type of the root:

Patient.name.given
The two expressions have the same outcome, but when evaluating the second, the evaluation will only produce results when used on data of type Patient. When resolving an identifier that is also the root of a FHIRPath expression, it is resolved as a type name first, and if it resolves to a type, it must resolve to the type of the context (or a supertype). Otherwise, it is resolved as a path on the context. If the identifier cannot be resolved, an error is raised.

Syntactically, FHIRPath defines identifiers as any sequence of characters consisting only of letters, digits, and underscores, beginning with a letter or underscore. Paths may use backticks to include characters in path parts that would otherwise be interpreted as keywords or operators, e.g.:

Message.`PID-1`
3.1. Collections
Collections are fundamental to FHIRPath, in that the result of every expression is a collection, even if that expression only results in a single element. This approach allows paths to be specified without having to care about the cardinality of any particular element, and is therefore ideally suited to graph traversal.

Within FHIRPath, a collection is:

Ordered - The order of items in the collection is important and is preserved through operations as much as possible. Operators and functions that do not preserve order will note that in their documentation.

Non-Unique - Duplicate elements are allowed within a collection. Some operations and functions, such as distinct() and the union operator | produce collections of unique elements, but in general, duplicate elements are allowed.

Indexed - Each item in a collection can be addressed by it’s index, i.e. ordinal position within the collection.

Unless specified otherwise by the underlying Object Model, the first item in a collection has index 0. Note that if the underlying model specifies that a collection is 1-based (the only reasonable alternative to 0-based collections), any collections generated from operations on the 1-based list are 0-based.

Countable - The number of items in a given collection can always be determined using the count() function

Note that the outcome of functions like children() and descendants() cannot be assumed to be in any meaningful order, and first(), last(), tail(), skip() and take() should not be used on collections derived from these paths. Note that some implementations may follow the logical order implied by the data model, and some may not, and some may be different depending on the underlying source. Implementations may decide to return an error if an attempt is made to perform an order-dependent operation on an unordered list.

3.2. Paths and polymorphic items
In the underlying representation of data, nodes may be typed and represent polymorphic items. Paths may either ignore the type of a node, and continue along the path or may be explicit about the expected node and filter the set of nodes by type before navigating down child nodes:

Observation.value.unit - all kinds of value
Observation.value.ofType(Quantity).unit - only values that are of type Quantity
The is operator can be used to determine whether or not a given value is of a given type:

Observation.value is Quantity // returns true if the value is of type Quantity
The as operator can be used to treat a value as a specific type:

Observation.value as Quantity // returns value as a Quantity if it is of type Quantity, and an empty result otherwise
The list of available types that can be passed as a parameter to the ofType() function and is and as operators is determined by the underlying data model. Within FHIRPath, they are just identifiers, either delimited or simple.

4. Expressions
FHIRPath expressions can consist of paths, literals, operators, and function invocations, and these elements can be chained together, so that the output of one operation or function is the input to the next. This is the core of the fluent syntactic style and allows complex paths and expressions to be built up from simpler components.

4.1. Literals
In addition to paths, FHIRPath expressions may contain literals, operators, and function invocations. FHIRPath supports the following types of literals:

Boolean: true, false
String: 'test string', 'urn:oid:3.4.5.6.7.8'
Integer: 0, 45
Decimal: 0.0, 3.14159265
Date: @2015-02-04 (@ followed by ISO8601 compliant date)
DateTime: @2015-02-04T14:34:28+09:00 (@ followed by ISO8601 compliant date/time)
Time: @T14:34:28 (@ followed by ISO8601 compliant time beginning with T, no timezone offset)
Quantity: 10 'mg', 4 days
For each type of literal, FHIRPath defines a named system type to allow operations and functions to be defined. For example, the multiplication operator (*) is defined for the numeric types Integer and Decimal, as well as the Quantity type. See the discussion on Models for a more detailed discussion of how these types are used within evaluation contexts.

4.1.1. Boolean
The Boolean type represents the logical Boolean values true and false. These values are used as the result of comparisons, and can be combined using logical operators such as and and or.

true
false
4.1.2. String
The String type represents string values up to 231-1 characters in length. String literals are surrounded by single-quotes and may use \-escapes to escape quotes and represent Unicode characters:

Escape	Character
\'

Single-quote

\"

Double-quote

\`

Backtick

\r

Carriage Return

\n

Line Feed

\t

Tab

\f

Form Feed

\\

Backslash

\uXXXX

Unicode character, where XXXX is the hexadecimal representation of the character

Note that Unicode is supported in both string literals and delimited identifiers.

'test string'
'urn:oid:3.4.5.6.7.8'
4.1.3. Integer
The Integer type represents whole numbers in the range -231 to 231-1.

0
45
-5
Note that the minus sign (-) in the representation of a negative integer is not part of the literal, it is the unary negation operator defined as part of FHIRPath syntax.

4.1.4. Decimal
The Decimal type represents real values in the range (-1028+1)/108 to (1028-1)/108 with a step size of 10-8. This range is defined based on a survey of decimal-value implementations and is based on the most useful lowest common denominator. Implementations can provide support for larger decimals and higher precision, but must provide at least the range and precision defined here. In addition, implementations should use fixed-precision decimal formats to ensure that decimal values are accurately represented.

0.0
3.14159265
Note that decimal literals cannot use exponential notation. There is enough additional complexity associated with enabling exponential notation that this is outside the scope of what FHIRPath is intended to support (namely graph traversal).

4.1.5. Date
The Date type represents date and partial date values in the range @0001-01-01 to @9999-12-31 with a 1 day step size.

The Date literal is a subset of [ISO8601]:

A date literal begins with an @

It uses the YYYY-MM-DD format, though month and day parts are optional

Week dates and ordinal dates are not allowed

Years must be present (-MM-DD is not a valid Date in FHIRPath)

Months must be present if a day is present

To specify a date and time together, see the description of DateTime below

The following examples illustrate the use of the Date literal:

@2014-01-25
@2014-01
@2014
Consult the formal grammar for more details.

4.1.6. Time
The Time type represents time-of-day and partial time-of-day values in the range @T00:00:00.0 to @T23:59:59.999 with a step size of 1 millisecond. This range is defined based on a survey of time implementations and is based on the most useful lowest common denominator. Implementations can provide support for higher precision, but must provide at least the range and precision defined here. Time values in FHIRPath do not have a timezone or timezone offset.

The Time literal uses a subset of [ISO8601]:

A time begins with a @T

It uses the Thh:mm:ss.fff format

The following examples illustrate the use of the Time literal:

@T12:00
@T14:30:14.559
Consult the formal grammar for more details.

4.1.7. DateTime
The DateTime type represents date/time and partial date/time values in the range @0001-01-01T00:00:00.0 to @9999-12-31T23:59:59.999 with a 1 millisecond step size. This range is defined based on a survey of datetime implementations and is based on the most useful lowest common denominator. Implementations can provide support for larger ranges and higher precision, but must provide at least the range and precision defined here.

The DateTime literal combines the Date and Time literals and is a subset of [ISO8601]:

A datetime literal begins with an @

It uses the YYYY-MM-DDThh:mm:ss.fff±hh:mm format

Timezone offset is optional, but if present the notation ±hh:mm is used (so must include both minutes and hours)

Z is allowed as a synonym for the zero (+00:00) UTC offset.

A T can be used at the end of any date (year, year-month, or year-month-day) to indicate a partial DateTime.

The following example illustrates the use of the DateTime literal:

@2014-01-25T14:30:14.559
@2014-01-25T14:30 // A partial DateTime with year, month, day, hour, and minute
@2014-03-25T // A partial DateTime with year, month, and day
@2014-01T // A partial DateTime with year and month
@2014T // A partial DateTime with only the year
The suffix T is allowed after a year, year-month, or year-month-day literal because without it, there would be no way to specify a partial DateTime with only a year, month, or day; the literal would always result in a Date value.

Consult the formal grammar for more details.

4.1.8. Quantity
The Quantity type represents quantities with a specified unit, where the value component is defined as a Decimal, and the unit element is represented as a String that is required to be a valid Unified Code for Units of Measure [UCUM] unit.

The Quantity literal is a number (integer or decimal), followed by a (single-quoted) string representing a valid Unified Code for Units of Measure [UCUM] unit. If the value literal is an Integer, it will be implicitly converted to a Decimal in the resulting Quantity value:

4.5 'mg'
100 '[degF]'
Note: When using [UCUM] units within FHIRPath, implementations shall use case-sensitive comparisons.

For date/time units, an alternative representation may be used (note that both a plural and singular version exist):

year/years, month/months, week/weeks, day/days, hour/hours, minute/minutes, second/seconds, millisecond/milliseconds

1 year
4 days
Note: Although [UCUM] identifies 'a' as 365.25 days, and 'mo' as 1/12 of a year, calculations involving durations shall round using calendar semantics as specified in [ISO8601].

4.2. Operators
Expressions can also contain operators, like those for mathematical operations and boolean logic:

Appointment.minutesDuration / 60 > 5
MedicationAdministration.wasNotGiven implies MedicationAdministration.reasonNotGiven.exists()
name.given | name.family // union of given and family names
'sir ' + name.given
Operators available in FHIRPath are covered in detail in the Operations section.

4.3. Function Invocations
Finally, FHIRPath supports the notion of functions, which all take a collection of values as input and produce another collection as output and may take parameters. For example:

(name.given | name.family).substring(0,4)
identifier.where(use = 'official')
Since all functions work on collections, constants will first be converted to a collection when functions are invoked on constants:

(4+5).count()
will return 1, since this is implicitly a collection with one constant number 9.

In general, functions in FHIRPath take collections as input and produce collections as output. This property, combined with the syntactic style of dot invocation enables functions to be chained together, creating a fluent-style syntax:

Patient.telecom.where(use = 'official').union(Patient.contact.telecom.where(use = 'official')).exists().not()
Throughout the function documentation, this input parameter is implicitly assumed, rather than explicitly documented in the function signature like the other arguments. For a complete listing of the functions defined in FHIRPath, refer to the Functions section.

4.4. Null and empty
There is no concept of null in FHIRPath. This means that when, in an underlying data object a member is null or missing, there will simply be no corresponding node for that member in the tree, e.g. Patient.name will return an empty collection (not null) if there are no name elements in the instance.

In expressions, the empty collection is represented as {}.

4.4.1. Propagation of empty results in expressions
FHIRPath functions and operators both propagate empty results, but the behavior is in general different when the argument to the function or operator expects a collection (e.g. select(), where() and | (union)) versus when the argument to the function or operator takes a single value as input (e.g. + and substring()).

For functions or operators that take a single values as input, this means in general if the input is empty, then the result will be empty as well. More specifically:

If a single-input operator or function operates on an empty collection, the result is an empty collection

If a single-input operator or function is passed an empty collection as an argument, the result is an empty collection

If any operand to a single-input operator or function is an empty collection, the result is an empty collection.

For operator or function arguments that expect collections, in general the empty collection is treated as any other collection would be. For example, the union (|) of an empty collection with some non-empty collection is that non-empty collection.

When functions or operators behave differently from these general principles, (for example the count() and empty() functions), this is clearly documented in the next sections.

4.5. Singleton Evaluation of Collections
In general, when a collection is passed as an argument to a function or operator that expects a single item as input, the collection is implicitly converted to a singleton as follows:

IF the collection contains a single node AND the node's value can be converted to the expected input type THEN
  The collection evaluates to the value of that single node
ELSE IF the collection contains a single node AND the expected input type is Boolean THEN
  The collection evaluates to true
ELSE IF the collection is empty THEN
  The collection evaluates to an empty collection
ELSE
  An error is raised
For example:

Patient.name.family + ', ' + Patient.name.given
If the Patient instance has a single name, and that name has a single given, then this will evaluate without any issues. However, if the Patient has multiple name elements, or the single name has multiple given elements, then it’s ambiguous which of the elements should be used as the input to the + operator, and the result is an error.

As another example:

Patient.active and Patient.gender and Patient.telecom
Assuming the Patient instance has an active value of true, a gender of female and a single telecom element, this expression will result in true. However, consider a different instance of Patient that has an active value of true, a gender of male, and multiple telecom elements, then this expression will result in an error because of the multiple telecom elements.

5. Functions
Functions are distinguished from path navigation names by the fact that they are followed by a () with zero or more parameters. With a few minor exceptions (e.g. the today() function), functions in FHIRPath always take a collection as input and produce another collection as output, even though these may be collections of just a single item.

Correspondingly, arguments to the functions can be any FHIRPath expression, though functions taking a single item as input require these expressions to evaluate to a collection containing a single item of a specific type. This approach allows functions to be chained, successively operating on the results of the previous function in order to produce the desired final result.

The following sections describe the functions supported in FHIRPath, detailing the expected types of parameters and type of collection returned by the function:

If the function expects a parameter to be a single value (e.g. item(index: Integer) and it is passed an argument that evaluates to a collection with multiple items, or to a collection with an item that is not of the required type (or cannot be converted to the required type), the evaluation of the expression will end and an error will be signaled to the calling environment.

If the function takes an expression as a parameter, the function will evaluate this parameter with respect to each of the items in the input collection. These expressions may refer to the special $this and $index elements, which represent the item from the input collection currently under evaluation, and its index in the collection, respectively. For example, in name.given.where($this > 'ba' and $this < 'bc') the where() function will iterate over each item in the input collection (elements named given) and $this will be set to each item when the expression passed to where() is evaluated.

Note that the bracket notation in function signatures indicates optional parameters, and is not part of the formal syntax of FHIRPath.

Note also that although all functions return collections, if a given function is defined to return a single element, the return type is simplified to just the type of the single element, rather than the list type.

5.1. Existence
5.1.1. empty() : Boolean
Returns true if the input collection is empty ({ }) and false otherwise.

5.1.2. exists([criteria : expression]) : Boolean
Returns true if the collection has any elements, and false otherwise. This is the opposite of empty(), and as such is a shorthand for empty().not(). If the input collection is empty ({ }), the result is false.

identifier.exists(use = 'official')
telecom.exists(system = 'phone' and use = 'mobile')
generalPractitioner.exists($this is Practitioner)
The function can also take an optional criteria to be applied to the collection prior to the determination of the exists. In this case, the function is shorthand for where(criteria).exists().

Note that a common term for this function is any.

5.1.3. all(criteria : expression) : Boolean
Returns true if for every element in the input collection, criteria evaluates to true. Otherwise, the result is false. If the input collection is empty ({ }), the result is true.

generalPractitioner.all($this is Practitioner)
5.1.4. allTrue() : Boolean
Takes a collection of Boolean values and returns true if all the items are true. If any items are false, the result is false. If the input is empty ({ }), the result is true.

The following example returns true if all of the components of the Observation have a value greater than 90 mm[Hg]:

Observation.select(component.value > 90 'mm[Hg]').allTrue()
5.1.5. anyTrue() : Boolean
Takes a collection of Boolean values and returns true if any of the items are true. If all the items are false, or if the input is empty ({ }), the result is false.

The following example returns true if any of the components of the Observation have a value greater than 90 mm[Hg]:

Observation.select(component.value > 90 'mm[Hg]').anyTrue()
5.1.6. allFalse() : Boolean
Takes a collection of Boolean values and returns true if all the items are false. If any items are true, the result is false. If the input is empty ({ }), the result is true.

The following example returns true if none of the components of the Observation have a value greater than 90 mm[Hg]:

Observation.select(component.value > 90 'mm[Hg]').allFalse()
5.1.7. anyFalse() : Boolean
Takes a collection of Boolean values and returns true if any of the items are false. If all the items are true, or if the input is empty ({ }), the result is false.

The following example returns true if any of the components of the Observation have a value that is not greater than 90 mm[Hg]:

Observation.select(component.value > 90 'mm[Hg]').anyFalse()
5.1.8. subsetOf(other : collection) : Boolean
Returns true if all items in the input collection are members of the collection passed as the other argument. Membership is determined using the = (Equals) (=) operation.

Conceptually, this function is evaluated by testing each element in the input collection for membership in the other collection, with a default of true. This means that if the input collection is empty ({ }), the result is true, otherwise if the other collection is empty ({ }), the result is false.

The following example returns true if the tags defined in any contained resource are a subset of the tags defined in the MedicationRequest resource:

MedicationRequest.contained.meta.tag.subsetOf(MedicationRequest.meta.tag)
5.1.9. supersetOf(other : collection) : Boolean
Returns true if all items in the collection passed as the other argument are members of the input collection. Membership is determined using the = (Equals) (=) operation.

Conceptually, this function is evaluated by testing each element in the other collection for membership in the input collection, with a default of true. This means that if the other collection is empty ({ }), the result is true, otherwise if the input collection is empty ({ }), the result is false.

The following example returns true if the tags defined in any contained resource are a superset of the tags defined in the MedicationRequest resource:

MedicationRequest.contained.meta.tag.supersetOf(MedicationRequest.meta.tag)
5.1.10. isDistinct() : Boolean
Returns true if all the items in the input collection are distinct. To determine whether two items are distinct, the = (Equals) (=) operator is used, as defined below.

Conceptually, this function is shorthand for a comparison of the count() of the input collection against the count() of the distinct() of the input collection:

X.count() = X.distinct().count()
This means that if the input collection is empty ({ }), the result is true.

5.1.11. distinct() : collection
Returns a collection containing only the unique items in the input collection. To determine whether two items are the same, the = (Equals) (=) operator is used, as defined below.

If the input collection is empty ({ }), the result is empty.

Note that the order of elements in the input collection is not guaranteed to be preserved in the result.

The following example returns the distinct list of tags on the given Patient:

Patient.meta.tag.distinct()
5.1.12. count() : Integer
Returns a collection with a single value which is the integer count of the number of items in the input collection. Returns 0 when the input collection is empty.

5.2. Filtering and projection
5.2.1. where(criteria : expression) : collection
Returns a collection containing only those elements in the input collection for which the stated criteria expression evaluates to true. Elements for which the expression evaluates to false or empty ({ }) are not included in the result.

If the input collection is emtpy ({ }), the result is empty.

The following example returns the list of telecom elements that have a use element with the value of 'official':

Patient.telecom.where(use = 'official')
5.2.2. select(projection: expression) : collection
Evaluates the projection expression for each item in the input collection. The result of each evaluation is added to the output collection. If the evaluation results in a collection with multiple items, all items are added to the output collection (collections resulting from evaluation of projection are flattened). This means that if the evaluation for an element results in the empty collection ({ }), no element is added to the result, and that if the input collection is empty ({ }), the result is empty as well.

Bundle.entry.select(resource as Patient)
This example results in a collection with only the patient resources from the bundle.

Bundle.entry.select((resource as Patient).telecom.where(system = 'phone'))
This example results in a collection with all the telecom elements with system of phone for all the patients in the bundle.

Patient.name.where(use = 'usual').select(given.first() + ' ' + family)
5.2.3. repeat(projection: expression) : collection
A version of select that will repeat the projection and add it to the output collection, as long as the projection yields new items (as determined by the = (Equals) (=) operator).

This function can be used to traverse a tree and selecting only specific children:

ValueSet.expansion.repeat(contains)
Will repeat finding children called contains, until no new nodes are found.

Questionnaire.repeat(item)
Will repeat finding children called item, until no new nodes are found.

Note that this is slightly different from:

Questionnaire.descendants().select(item)
which would find any descendants called item, not just the ones nested inside other item elements.

The order of items returned by the repeat() function is undefined.

5.2.4. ofType(type : TypeInfo) : collection
Returns a collection that contains all items in the input collection that are of the given type or a subclass thereof. If the input collection is empty ({ }), the result is empty. The type argument is an identifier that must resolve to the name of a type in a model. For implementations with compile-time typing, this requires special-case handling when processing the argument to treat is a type specifier rather than an identifier expression:

Bundle.entry.resource.ofType(Patient)
5.3. Subsetting
5.3.1. [ index : Integer ] : collection
The indexer operation returns a collection with only the index-th item (0-based index). If the input collection is empty ({ }), or the index lies outside the boundaries of the input collection, an empty collection is returned.

Note: Unless specified otherwise by the underlying Object Model, the first item in a collection has index 0. Note that if the underlying model specifies that a collection is 1-based (the only reasonable alternative to 0-based collections), any collections generated from operations on the 1-based list are 0-based.

The following example returns the 0th name of the Patient:

Patient.name[0]
5.3.2. single() : collection
Will return the single item in the input if there is just one item. If the input collection is empty ({ }), the result is empty. If there are multiple items, an error is signaled to the evaluation environment. This function is useful for ensuring that an error is returned if an assumption about cardinality is violated at run-time.

The following example returns the name of the Patient if there is one. If there are no names, an empty collection, and if there are multiple names, an error is signaled to the evaluation environment:

Patient.name.single()
5.3.3. first() : collection
Returns a collection containing only the first item in the input collection. This function is equivalent to item[0], so it will return an empty collection if the input collection has no items.

5.3.4. last() : collection
Returns a collection containing only the last item in the input collection. Will return an empty collection if the input collection has no items.

5.3.5. tail() : collection
Returns a collection containing all but the first item in the input collection. Will return an empty collection if the input collection has no items, or only one item.

5.3.6. skip(num : Integer) : collection
Returns a collection containing all but the first num items in the input collection. Will return an empty collection if there are no items remaining after the indicated number of items have been skipped, or if the input collection is empty. If num is less than or equal to zero, the input collection is simply returned.

5.3.7. take(num : Integer) : collection
Returns a collection containing the first num items in the input collection, or less if there are less than num items. If num is less than or equal to 0, or if the input collection is empty ({ }), take returns an empty collection.

5.3.8. intersect(other: collection) : collection
Returns the set of elements that are in both collections. Duplicate items will be eliminated by this function. Order of items is not guaranteed to be preserved in the result of this function.

5.3.9. exclude(other: collection) : collection
Returns the set of elements that are not in the other collections. Duplicate items will not be eliminated by this function, and order will be preserved.

e.g. Patient.children().exclude(name|birthDate) would return all the properties of the Patient except for the name and birthDate.

5.4. Combining
5.4.1. union(other : collection)
Merge the two collections into a single collection, eliminating any duplicate values (using = (Equals) (=) to determine equality). Unioning an empty collection to a non-empty collection will return the non-empty collection with duplicates eliminated. There is no expectation of order in the resulting collection.

In other words, this function returns the distinct list of elements from both inputs. For example, consider two lists of integers A: 1, 1, 2, 3 and B: 2, 3:

A union B // 1, 2, 3
A union { } // 1, 2, 3
This function can also be invoked using the | operator.

a.union(b)
is synonymous with

a | b
5.4.2. combine(other : collection) : collection
Merge the input and other collections into a single collection without eliminating duplicate values. Combining an empty collection with a non-empty collection will return the non-empty collection. There is no expectation of order in the resulting collection.

5.5. Conversion
The functions in this section operate on collections with a single item. If there is more than one item, the evaluation of the expression will end and signal an error to the calling environment.

Note that although all functions return collections, if a given function is defined to return a single element, the return type in the description of the function is simplified to just the type of the single element, rather than the list type.

The following table lists the possible conversions supported, and whether the conversion is implicit or explicit:

From\To	Boolean	Integer	Decimal	Quantity	String	Date	DateTime	Time
Boolean

N/A

Explicit

Explicit

-

Explicit

-

-

-

Integer

Explicit

N/A

Implicit

Implicit

Explicit

-

-

-

Decimal

Explicit

-

N/A

Implicit

Explicit

-

-

-

Quantity

-

-

-

N/A

Explicit

-

-

-

String

Explicit

Explicit

Explicit

Explicit

N/A

Explicit

Explicit

Explicit

Date

-

-

-

-

Explicit

N/A

Implicit

-

DateTime

-

-

-

-

Explicit

Explicit

N/A

-

Time

-

-

-

-

Explicit

-

-

N/A

Implicit conversion is performed when an operator or function is used with a compatible type. For example:

5 + 10.0
In the above expression, the addition operator expects either two Integers, or two Decimals, so implicit conversion is used to convert the integer to a decimal, resulting in decimal addition.

To use these functions over a collection with multiple items, one may use filters like where() and select():

Patient.name.given.select(substring(0))
This example returns a collection containing the first character of all the given names for a patient.

5.5.1. iif(criterion: expression, true-result: collection [, otherwise-result: collection]) : collection
If criterion is true, the function returns the value of true-result parameter.

If criterion is false or an empty collection, the function returns otherwise-result, unless the optional otherwise-result is not given, in which case the function returns an empty collection.

Note that short-circuit behavior is expected in this function. In other words, true-result should only be evaluated if the criterion evaluates to true, and otherwise-result should only be evaluated otherwise. For implementations, this means delaying evaluation of the arguments.

5.5.2. convertsToBoolean() : Boolean
If the input collection contains a single item, this function will return true if:

the item is a Boolean

the item is an Integer and is convertible to a Boolean using one of the possible integer representations of Boolean values

the item is a Decimal and is convertible to a Boolean using one of the possible decimal representations of Boolean values

the item is a String and is convertible to a Boolean using one of the possible string representations of Boolean values

If the item is not one of the above types, or the item is a String or Integer, but is not one of the possible values convertible to a Boolean, the result is false.

Possible values for Integer, Decimal, and String are described in the toBoolean() function.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

If the input collection is empty, the result is empty.

5.5.3. toBoolean() : Boolean
If the input collection contains a single item, this function will return a single boolean if:

the item is a Boolean

the item is an Integer and is convertible to a Boolean using one of the possible integer representations of Boolean values

the item is a Decimal and is convertible to a Boolean using one of the possible decimal representation of Boolean values

the item is a String and is convertible to a Boolean using one of the possible string representations of Boolean values

If the item is not one the above types, or the item is a String or Integer, but is not one of the possible values convertible to a Boolean, the result is empty.

If the item is a String, but the string is not convertible to a boolean (using one of the possible string representations of Boolean values), the result is empty.

The following table describes the possible values convertible to an Boolean:

Type	Representation	Result
String

'true', 't', 'yes', 'y', '1', '1.0'

true

'false', 'f', 'no', 'n', '0', '0.0'

false

Integer

1

true

0

false

Decimal

1.0

true

0.0

false

Note for the purposes of string representations, case is ignored (so that both 'T' and 't' are considered true).

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

If the input collection is empty, the result is empty.

5.5.4. convertsToInteger() : Boolean
If the input collection contains a single item, this function will return true if:

the item is an Integer

the item is a String and is convertible to an Integer

the item is a Boolean

If the item is not one of the above types, or the item is a String, but is not convertible to an Integer (using the regex format (\\+|-)?\d+), the result is false.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

If the input collection is empty, the result is empty.

5.5.5. toInteger() : Integer
If the input collection contains a single item, this function will return a single integer if:

the item is an Integer

the item is a String and is convertible to an integer

the item is a Boolean, where true results in a 1 and false results in a 0.

If the item is not one the above types, the result is empty.

If the item is a String, but the string is not convertible to an integer (using the regex format (\\+|-)?\d+), the result is empty.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

If the input collection is empty, the result is empty.

5.5.6. convertsToDate() : Boolean
If the input collection contains a single item, this function will return true if:

the item is a Date

the item is a DateTime

the item is a String and is convertible to a Date

If the item is not one of the above types, or is not convertible to a Date (using the format YYYY-MM-DD), the result is false.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

If the input collection is empty, the result is empty.

5.5.7. toDate() : Date
If the input collection contains a single item, this function will return a single date if:

the item is a Date

the item is a DateTime

the item is a String and is convertible to a Date

If the item is not one of the above types, the result is empty.

If the item is a String, but the string is not convertible to a Date (using the format YYYY-MM-DD), the result is empty.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

If the input collection is empty, the result is empty.

5.5.8. convertsToDateTime() : Boolean
If the input collection contains a single item, this function will return true if:

the item is a DateTime

the item is a Date

the item is a String and is convertible to a DateTime

If the item is not one of the above types, or is not convertible to a DateTime (using the format YYYY-MM-DDThh:mm:ss.fff(+/-)hh:mm), the result is false.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

If the input collection is empty, the result is empty.

5.5.9. toDateTime() : DateTime
If the input collection contains a single item, this function will return a single datetime if:

the item is a DateTime

the item is a Date, in which case the result is a DateTime with the year, month, and day of the Date, and the time components empty (not set to zero)

the item is a String and is convertible to a DateTime

If the item is not one of the above types, the result is empty.

If the item is a String, but the string is not convertible to a DateTime (using the format YYYY-MM-DDThh:mm:ss.fff(+/-)hh:mm), the result is empty.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

If the input collection is empty, the result is empty.

5.5.10. convertsToDecimal() : Boolean
If the input collection contains a single item, this function will true if:

the item is an Integer or Decimal

the item is a String and is convertible to a decimal

the item is a Boolean

If the item is not one of the above types, or is not convertible to a decimal (using the regex format (\\+|-)?\d+('.'\d+)?), the result is false.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

If the input collection is empty, the result is empty.

5.5.11. toDecimal() : Decimal
If the input collection contains a single item, this function will return a single decimal if:

the item is an Integer or Decimal

the item is a String and is convertible to a decimal

the item is a Boolean, where true results in a 1.0 and false results in a 0.0.

If the item is not one of the above types, the result is empty.

If the item is a String, but the string is not convertible to a decimal (using the regex format (\\+|-)?\d+('.' \d+)?), the result is empty.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

If the input collection is empty, the result is empty.

5.5.12. convertsToQuantity([unit : String]) : Boolean
If the input collection contains a single item, this function will return true if:

the item is an Integer, Decimal, or Quantity

the item is a String that is convertible to a quantity

the item is a Boolean

If the item is not one of the above types, or is not convertible to a quantity (using the regex format (?'value'(\\+|-)?\d+(\.\d+)?)\s*'(?'unit'[^&#39;]+)')|(?'time'[a-zA-Z]+?), the result is false.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

If the input collection is empty, the result is empty.

If the unit argument is provided, it must be the string representation of a UCUM code, and is used to determine whether the input quantity can be converted to the given unit, according to the unit conversion rules specified by UCUM. If the input quantity can be converted, the result is true, otherwise, the result is false.

Note: Implementations are not required to support a complete UCUM implementation, and are free to return false when the unit argument is used and it is different than the input quantity unit.

5.5.13. toQuantity([unit : String]) : Quantity
If the input collection contains a single item, this function will return a single quantity if:

the item is an Integer, or Decimal, where the resulting quantity will have the default unit ('1')

the item is a Quantity

the item is a String and is convertible to a quantity

the item is a Boolean, where true results in the quantity 1.0 '1', and false results in the quantity 0.0 '1'

If the item is not one of the above types, the result is empty.

If the item is a String, but the string is not convertible to a quantity (using the regex format (?'value'(\\+|-)?\d+(\.\d+)?)\s*'(?'unit'[^&#39;]+)')|(?'time'[a-zA-Z]+?), the result is empty.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

If the input collection is empty, the result is empty.

If the unit argument is provided, it must be the string representation of a UCUM code, and is used to determine whether the input quantity can be converted to the given unit, according to the unit conversion rules specified by UCUM. If the input quantity can be converted, the result is true, otherwise, the result is false.

Note: Implementations are not required to support a complete UCUM implementation, and are free to return empty ({ }) when the unit argument is used and it is different than the input quantity unit.

5.5.14. convertsToString() : String
If the input collection contains a single item, this function will return true if:

the item is a String

the item is an Integer, Decimal, Date, Time, or DateTime

the item is a Boolean

the item is a Quantity

If the item is not one of the above types, the result is false.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

If the input collection is empty, the result is empty.

5.5.15. toString() : String
If the input collection contains a single item, this function will return a single String if:

the item in the input collection is a String

the item in the input collection is an Integer, Decimal, Date, Time, DateTime, or Quantity the output will contain its String representation

the item is a Boolean, where true results in 'true' and false in 'false'.

If the item is not one of the above types, the result is false.

The String representation uses the following formats:

Type	Representation
Boolean

true or false

Integer

(\\+|-)?\d+

Decimal

(\\+|-)?\d+(.\d+)?

Quantity

(\\+|-)?\d+(.\d+)? '.*'

Date

YYYY-MM-DD

DateTime

YYYY-MM-DDThh:mm:ss.fff(+/-)hh:mm

Time

hh:mm:ss.fff(+/-)hh:mm

Note that for partial dates and times, the result will only be specified to the level of precision in the value being converted.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

If the input collection is empty, the result is empty.

5.5.16. convertsToTime() : Boolean
If the input collection contains a single item, this function will return true if:

the item is a Time

the item is a String and is convertible to a Time

If the item is not one of the above types, or is not convertible to a Time (using the format hh:mm:ss.fff(+/-)hh:mm), the result is false.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

If the input collection is empty, the result is empty.

5.5.17. toTime() : Time
If the input collection contains a single item, this function will return a single time if:

the item is a Time

the item is a String and is convertible to a Time

If the item is not one of the above types, the result is empty.

If the item is a String, but the string is not convertible to a Time (using the format hh:mm:ss.fff(+/-)hh:mm), the result is empty.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

If the input collection is empty, the result is empty.

5.6. String Manipulation
The functions in this section operate on collections with a single item. If there is more than one item, or an item that is not a String, the evaluation of the expression will end and signal an error to the calling environment.

Note that although all functions return collections, if a given function is defined to return a single element, the return type in the description of the function is simplified to just the type of the single element, rather than the list type.

5.6.1. indexOf(substring : String) : Integer
Returns the 0-based index of the first position substring is found in the input string, or -1 if it is not found.

If substring is an empty string (''), the function returns 0.

If the input or substring is empty ({ }), the result is empty ({ }).

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

'abcdefg'.indexOf('bc') // 1
'abcdefg'.indexOf('x') // -1
'abcdefg'.indexOf('abcdefg') // 0
5.6.2. substring(start : Integer [, length : Integer]) : String
Returns the part of the string starting at position start (zero-based). If length is given, will return at most length number of characters from the input string.

If start lies outside the length of the string, the function returns empty ({ }). If there are less remaining characters in the string than indicated by length, the function returns just the remaining characters.

If the input or start is empty, the result is empty.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

'abcdefg'.substring(3) // 'defg'
'abcdefg'.substring(1, 2) // 'bc'
'abcdefg'.substring(6, 2) // 'g'
'abcdefg'.substring(7, 1) // { }
5.6.3. startsWith(prefix : String) : Boolean
Returns true when the input string starts with the given prefix.

If prefix is the empty string (''), the result is true.

If the input collection is empty, the result is empty.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

'abcdefg'.startsWith('abc') // true
'abcdefg'.startsWith('xyz') // false
5.6.4. endsWith(suffix : String) : Boolean
Returns true when the input string ends with the given suffix.

If suffix is the empty string (''), the result is true.

If the input collection is empty, the result is empty.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

'abcdefg'.endsWith('efg') // true
'abcdefg'.ednsWith('abc') // false
5.6.5. contains(substring : String) : Boolean
Returns true when the given substring is a substring of the input string.

If substring is the empty string (''), the result is true.

If the input collection is empty, the result is empty.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

'abc'.contains('b') // true
'abc'.contains('bc') // true
'abc'.contains('d') // false
Note: The .contains() function described here is a string function that looks for a substring in a string. This is different than the contains operator, which is a list operator that looks for an element in a list.

5.6.6. upper() : String
Returns the input string with all characters converted to upper case.

If the input collection is empty, the result is empty.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

'abcdefg'.upper() // 'ABCDEFG'
'AbCdefg'.upper() // 'ABCDEFG'
5.6.7. lower() : String
Returns the input string with all characters converted to lower case.

If the input collection is empty, the result is empty.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

'ABCDEFG'.lower() // 'abcdefg'
'aBcDEFG'.lower() // 'abcdefg'
5.6.8. replace(pattern : String, substitution : String) : String
Returns the input string with all instances of pattern replaced with substitution. If the substitution is the empty string ('‘), instances of `pattern are removed from the result. If pattern is the empty string (’'`), every character in the input string is surrounded by the substitution, e.g. 'abc'.replace('','x') becomes 'xaxbxcx'.

If the input collection, pattern, or substitution are empty, the result is empty ({ }).

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

'abcdefg'.replace('cde', '123') // 'ab123fg'
'abcdefg'.replace('cde', '') // 'abfg'
'abc'.replace('', 'x') // 'xaxbxcx'
5.6.9. matches(regex : String) : Boolean
Returns true when the value matches the given regular expression. Regular expressions should function consistently, regardless of any culture- and locale-specific settings in the environment, should be case-sensitive, use 'single line' mode and allow Unicode characters.

If the input collection or regex are empty, the result is empty ({ }).

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

5.6.10. replaceMatches(regex : String, substitution: String) : String
Matches the input using the regular expression in regex and replaces each match with the substitution string. The substitution may refer to identified match groups in the regular expression.

If the input collection, regex, or substituion are empty, the result is empty ({ }).

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

This example of replaceMatches() will convert a string with a date formatted as MM/dd/yy to dd-MM-yy:

'11/30/1972'.replace('\\b(?<month>\\d{1,2})/(?<day>\\d{1,2})/(?<year>\\d{2,4})\\b',
       '${day}-${month}-${year}')
Note: Platforms will typically use native regular expression implementations. These are typically fairly similar, but there will always be small differences. As such, FHIRPath does not prescribe a particular dialect, but recommends the use of the [PCRE] flavor as the dialect most likely to be broadly supported and understood.

5.6.11. length() : Integer
Returns the length of the input string. If the input collection is empty ({ }), the result is empty.

5.6.12. toChars() : collection
Returns the list of characters in the input string. If the input collection is empty ({ }), the result is empty.

'abc'.toChars() // { 'a', 'b', 'c' }
5.7. Math
Note: the contents of this section are Standard for Trial Use (STU)

The functions in this section operate on collections with a single item. Unless otherwise noted, if there is more than one item, or the item is not compatible with the expected type, the evaluation of the expression will end and signal an error to the calling environment.

Note also that although all functions return collections, if a given function is defined to return a single element, the return type in the description of the function is simplified to just the type of the single element, rather than the list type.

5.7.1. abs() : Integer | Decimal | Quantity
Returns the absolute value of the input. When taking the absolute value of a quantity, the unit is unchanged.

If the input collection is empty, the result is empty.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

(-5).abs() // 5
(-5.5).abs() // 5.5
(-5.5 'mg').abs() // 5.5 'mg'
5.7.2. ceiling() : Integer
Returns the first integer greater than or equal to the input.

If the input collection is empty, the result is empty.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

1.ceiling() // 1
1.1.ceiling() // 2
(-1.1).ceiling() // -1
5.7.3. exp() : Decimal
Returns e raised to the power of the input.

If the input collection contains an Integer, it will be implicitly converted to a Decimal and the result will be a Decimal.

If the input collection is empty, the result is empty.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

0.exp() // 1
(-0.0).exp() // 1
5.7.4. floor() : Integer
Returns the first integer less than or equal to the input.

If the input collection is empty, the result is empty.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

1.floor() // 1
2.1.floor() // 2
(-2.1).floor() // -3
5.7.5. ln() : Decimal
Returns the natural logarithm of the input (i.e. the logarithm base e).

When used with an Integer, it will be implicitly converted to a Decimal.

If the input collection is empty, the result is empty.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

1.ln() // 0.0
1.0.ln() // 0.0
5.7.6. log(base : Decimal) : Decimal
Returns the logarithm base base of the input number.

When used with Integers, the arguments will be implicitly converted to Decimal.

If base is empty, the result is empty.

If the input collection is empty, the result is empty.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

16.log(2) // 4.0
100.0.log(10.0) // 2.0
5.7.7. power(exponent : Integer | Decimal) : Integer | Decimal
Raises a number to the exponent power. If this function is used with Integers, the result is an Integer. If the function is used with Decimals, the result is a Decimal. If the function is used with a mixture of Integer and Decimal, the Integer is implicitly converted to a Decimal and the result is a Decimal.

If the power cannot be represented (such as the -1 raised to the 0.5), the result is empty.

If the input is empty, or exponent is empty, the result is empty.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

2.power(3) // 8
2.5.power(2) // 6.25
(-1).power(0.5) // empty ({ })
5.7.8. round([precision : Integer]) : Decimal
Rounds the decimal to the nearest whole number using a traditional round (i.e. 0.5 or higher will round to 1). If specified, the precision argument determines the decimal place at which the rounding will occur.

If the input collection contains a single item of type Integer, it will be implicitly converted to a Decimal.

If the input collection is empty, the result is empty.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

1.round() // 1
3.14159.round(3) // 3.142
5.7.9. sqrt() : Decimal
Returns the square root of the input number as a Decimal.

If the square root cannot be represented (such as the square root of -1), the result is empty.

If the input collection is empty, the result is empty.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

Note that this function is equivalent to raising a number of the power of 0.5 using the power() function.

81.sqrt() // 9.0
(-1).sqrt() // empty
5.7.10. truncate() : Integer
Returns the integer portion of the input.

If the input collection is empty, the result is empty.

If the input collection contains multiple items, the evaluation of the expression will end and signal an error to the calling environment.

101.truncate() // 101
1.00000001.truncate() // 1
(-1.56).truncate() // -1
5.8. Tree navigation
5.8.1. children() : collection
Returns a collection with all immediate child nodes of all items in the input collection. Note that the ordering of the children is undefined and using functions like first() on the result may return different results on different platforms.

5.8.2. descendants() : collection
Returns a collection with all descendant nodes of all items in the input collection. The result does not include the nodes in the input collection themselves. This function is a shorthand for repeat(children()). Note that the ordering of the children is undefined and using functions like first() on the result may return different results on different platforms.

Note: Many of these functions will result in a set of nodes of different underlying types. It may be necessary to use ofType() as described in the previous section to maintain type safety. See Type safety and strict evaluation for more information about type safe use of FHIRPath expressions.

5.9. Utility functions
5.9.1. trace(name : String [, projection: Expression]) : collection
Adds a String representation of the input collection to the diagnostic log, using the parameter name as the name in the log. This log should be made available to the user in some appropriate fashion. Does not change the input, so returns the input collection as output.

If the projection argument is used, the trace would log the result of evaluating the project expression on the input, but still return the input to the trace function unchanged.

contained.where(criteria).trace('unmatched', id).empty()
The above example traces only the id elements of the result of the where.

5.9.2. Current date and time functions
The following functions return the current date and time. The timestamp that these functions use is an implementation decision, and implementations should consider providing options appropriate for their environment. In the simplest case, the local server time is used as the timestamp for these function.

To ensure deterministic evaluation, these operators should return the same value regardless of how many times they are evaluated within any given expression (i.e. now() should always return the same DateTime in a given expression, timeOfDay() should always return the same Time in a given expression, and today() should always return the same Date in a given expression.)

now() : DateTime
Returns the current date and time, including timezone offset.

timeOfDay() : Time
Returns the current time.

today() : Date
Returns the current date.

6. Operations
Operators are allowed to be used between any kind of path expressions (e.g. expr op expr). Like functions, operators will generally propagate an empty collection in any of their operands. This is true even when comparing two empty collections using the equality operators, e.g.

{} = {}
true > {}
{} != 'dummy'
all result in {}.

6.1. Equality
6.1.1. = (Equals)
Returns true if the left collection is equal to the right collection:

As noted above, if either operand is an empty collection, the result is an empty collection. Otherwise:

If both operands are collections with a single item, they must be of the same type, and:

For primitives:

String: comparison is based on Unicode values

Integer: values must be exactly equal

Decimal: values must be equal, trailing zeroes after the decimal are ignored

Boolean: values must be the same

Date: must be exactly the same

DateTime: must be exactly the same, respecting the timezone offset (though +00:00 = -00:00 = Z)

Time: must be exactly the same

For complex types, equality requires all child properties to be equal, recursively.

If both operands are collections with multiple items:

Each item must be equal

Comparison is order dependent

Otherwise, equals returns false.

Note that this implies that if the collections have a different number of items to compare, the result will be false.

Typically, this operator is used with single fixed values as operands. This means that Patient.telecom.system = 'phone' will result in an error if there is more than one telecom with a use. Typically, you’d want Patient.telecom.where(system = 'phone')

If one or both of the operands is the empty collection, this operation returns an empty collection.

When comparing quantities for equality, the dimensions of each quantity must be the same, but not necessarily the unit. For example, units of 'cm' and 'm' can be compared, but units of 'cm2' and 'cm' cannot. The unit of the result will be the most granular unit of either input. Attempting to operate on quantities with invalid units will result in empty ({ }).

Implementations are not required to fully support operations on units, but they must at least respect units, recognizing when units differ.

Implementations that do support units SHALL do so as specified by [UCUM].

Note: Although [UCUM] identifies 'a' as 365.25 days, and 'mo' as 1/12 of a year, calculations involving durations shall round using calendar semantics as specified in [ISO8601]. For comparisons involving durations (where no anchor to a calendar is available), the duration of a year is 365 days, and the duration of a month is 30 days.

For Date, DateTime and Time equality, the comparison is performed by considering each precision in order, beginning with years (or hours for time values), and respecting timezone offsets. If the values are the same, comparison proceeds to the next precision; if the values are different, the comparison stops and the result is false. If one input has a value for the precision and the other does not, the comparison stops and the result is empty ({ }); if neither input has a value for the precision, or the last precision has been reached, the comparison stops and the result is true. For the purposes of comparison, seconds and milliseconds are considered a single precision using a decimal, with decimal equality semantics.

For example:

@2012 = @2012 // returns true
@2012 = @2013 // returns false
@2012-01 = @2012 // returns empty ({ })
@2012-01-01T10:30 = @2012-01-01T10:30 // returns true
@2012-01-01T10:30 = @2012-01-01T10:31 // returns false
@2012-01-01T10:30:31 = @2012-01-01T10:30 // returns empty ({ })
@2012-01-01T10:30:31.0 = @2012-01-01T10:30:31 // returns true
@2012-01-01T10:30:31.1 = @2012-01-01T10:30:31 // returns false
For DateTime values that do not have a timezone offsets, whether or not to provide a default timezone offset is a policy decision. In the simplest case, no default timezone offset is provided, but some implementations may use the client’s or the evaluating system’s timezone offset.

To support comparison of DateTime values, either both values have no timezone offset specified, or both values are converted to a common timezone offset. The timezone offset to use is an implementation decision. In the simplest case, it’s the timezone offset of the local server. The following examples illustrate expected behavior:

@2017-11-05T01:30:00.0-04:00 > @2017-11-05T01:15:00.0-05:00 // false
@2017-11-05T01:30:00.0-04:00 < @2017-11-05T01:15:00.0-05:00 // true
@2017-11-05T01:30:00.0-04:00 = @2017-11-05T01:15:00.0-05:00 // false
@2017-11-05T01:30:00.0-04:00 = @2017-11-05T00:30:00.0-05:00 // true
Additional functions to support more sophisticated timezone offset comparison (such as .toUTC()) may be defined in a future version.

6.1.2. ~ (Equivalent)
Returns true if the collections are the same. In particular, comparing empty collections for equivalence { } ~ { } will result in true.

If both operands are collections with a single item, they must be of the same type, and:

For primitives

String: the strings must be the same, ignoring case and locale, and normalizing whitespace (see String Equivalence for more details).

Integer: exactly equal

Decimal: values must be equal, comparison is done on values rounded to the precision of the least precise operand. Trailing zeroes after the decimal are ignored in determining precision.

Date, DateTime and Time: values must be equal, except that if the input values have different levels of precision, the comparison returns false, not empty ({ }).

Boolean: the values must be the same

For complex types, equivalence requires all child properties to be equivalent, recursively.

If both operands are collections with multiple items:

Each item must be equivalent

Comparison is not order dependent

Note that this implies that if the collections have a different number of items to compare, or if one input is a value and the other is empty ({ }), the result will be false.

When comparing quantities for equivalence, the dimensions of each quantity must be the same, but not necessarily the unit. For example, units of 'cm' and 'm' can be compared, but units of 'cm2' and 'cm' cannot. The unit of the result will be the most granular unit of either input. Attempting to operate on quantities with invalid units will result in false.

Implementations are not required to fully support operations on units, but they must at least respect units, recognizing when units differ.

Implementations that do support units SHALL do so as specified by [UCUM].

Note: Although [UCUM] identifies 'a' as 365.25 days, and 'mo' as 1/12 of a year, calculations involving durations shall round using calendar semantics as specified in [ISO8601]. For comparisons involving durations (where no anchor to a calendar is available), the duration of a year is 365 days, and the duration of a month is 30 days.

For Date, DateTime and Time equivalence, the comparison is the same as for equality, with the exception that if the input values have different levels of precision, the result is false, rather than empty ({ }). As with equality, the second and millisecond precisions are considered a single precision using a decimal, with decimal equivalence semantics.

For example:

@2012 ~ @2012 // returns true
@2012 ~ @2013 // returns false
@2012-01 ~ @2012 // returns false as well
@2012-01-01T10:30 ~ @2012-01-01T10:30 // returns true
@2012-01-01T10:30 ~ @2012-01-01T10:31 // returns false
@2012-01-01T10:30:31 ~ @2012-01-01T10:30 // returns false as well
@2012-01-01T10:30:31.0 ~ @2012-01-01T10:30:31 // returns true
@2012-01-01T10:30:31.1 ~ @2012-01-01T10:30:31 // returns false
String Equivalence
For strings, equivalence returns true if the strings are the same value while ignoring case and locale, and normalizing whitespace. Normalizing whitespace means that all whitespace characters are treated as equivalent, with whitespace characters as defined in the Whitespace lexical category.

6.1.3. != (Not Equals)
The converse of the equals operator.

6.1.4. !~ (Not Equivalent)
The converse of the equivalent operator.

6.2. Comparison
The comparison operators are defined for strings, integers, decimals, quantities, dates, datetimes and times.

If one or both of the arguments is an empty collection, a comparison operator will return an empty collection.

Both arguments must be collections with single values, and the evaluator will throw an error if either collection has more than one item.

Both arguments must be of the same type, and the evaluator will throw an error if the types differ.

When comparing integers and decimals, the integer will be converted to a decimal to make comparison possible.

String ordering is strictly lexical and is based on the Unicode value of the individual characters.

When comparing quantities, the dimensions of each quantity must be the same, but not necessarily the unit. For example, units of 'cm' and 'm' can be compared, but units of 'cm2' and 'cm' cannot. The unit of the result will be the most granular unit of either input. Attempting to operate on quantities with invalid units will result in empty ({ }).

Implementations are not required to fully support operations on units, but they must at least respect units, recognizing when units differ.

Implementations that do support units SHALL do so as specified by [UCUM].

For partial Date, DateTime, and Time values, the comparison is performed by comparing the values at each precision, beginning with years, and proceeding to the finest precision specified in either input, and respecting timezone offsets. If one value is specified to a different level of precision than the other, the result is empty ({ }) to indicate that the result of the comparison is unknown. As with equality and equivalence, the second and millisecond precisions are considered a single precision using a decimal, with decimal comparison semantics.

See the Equals operator for discussion on respecting timezone offsets in comparison operations.

6.2.1. > (Greater Than)
The greater than operator (>) returns true if the first operand is strictly greater than the second. The operands must be of the same type, or convertible to the same type using an implicit conversion.

10 > 5 // true
10 > 5.0 // true; note the 10 is converted to a decimal to perform the comparison
'abc' > 'ABC' // true
4 'm' > 4 'cm' // true (or { } if the implementation does not support unit conversion)
@2018-03-01 > @2018-01-01 // true
@2018-03 > @2018-03-01 // empty ({ })
@2018-03-01T10:30:00 > @2018-03-01T10:00:00 // true
@2018-03-01T10 > @2018-03-01T10:30 // empty ({ })
@2018-03-01T10:30:00 > @2018-03-01T10:30:00.0 // false
@T10:30:00 > @T10:00:00 // true
@T10 > @T10:30 // empty ({ })
@T10:30:00 > @T10:30:00.0 // false
6.2.2. < (Less Than)
The less than operator (<) returns true if the first operand is strictly less than the second. The operands must be of the same type, or convertible to the same type using implicit conversion.

10 < 5 // false
10 < 5.0 // false; note the 10 is converted to a decimal to perform the comparison
'abc' < 'ABC' // false
4 'm' < 4 'cm' // false (or { } if the implementation does not support unit conversion)
@2018-03-01 < @2018-01-01 // false
@2018-03 < @2018-03-01 // empty ({ })
@2018-03-01T10:30:00 < @2018-03-01T10:00:00 // false
@2018-03-01T10 < @2018-03-01T10:30 // empty ({ })
@2018-03-01T10:30:00 < @2018-03-01T10:30:00.0 // false
@T10:30:00 < @T10:00:00 // false
@T10 < @T10:30 // empty ({ })
@T10:30:00 < @T10:30:00.0 // false
6.2.3. <= (Less or Equal)
The less or equal operator (<=) returns true if the first operand is less than or equal to the second. The operands must be of the same type, or convertible to the same type using implicit conversion.

10 <= 5 // true
10 <= 5.0 // true; note the 10 is converted to a decimal to perform the comparison
'abc' <= 'ABC' // true
4 'm' <= 4 'cm' // false (or { } if the implementation does not support unit conversion)
@2018-03-01 <= @2018-01-01 // false
@2018-03 <= @2018-03-01 // empty ({ })
@2018-03-01T10:30:00 <= @2018-03-01T10:00:00 // false
@2018-03-01T10 <= @2018-03-01T10:30 // empty ({ })
@2018-03-01T10:30:00 <= @2018-03-01T10:30:00.0 // true
@T10:30:00 <= @T10:00:00 // false
@T10 <= @T10:30 // empty ({ })
@T10:30:00 <= @T10:30:00.0 // true
6.2.4. >= (Greater or Equal)
The greater or equal operator (>=) returns true if the first operand is greater than or equal to the second. The operands must be of the same type, or convertible to the same type using implicit conversion.

10 >= 5 // false
10 >= 5.0 // false; note the 10 is converted to a decimal to perform the comparison
'abc' >= 'ABC' // false
4 'm' >= 4 'cm' // true (or { } if the implementation does not support unit conversion)
@2018-03-01 >= @2018-01-01 // true
@2018-03 >= @2018-03-01 // empty ({ })
@2018-03-01T10:30:00 >= @2018-03-01T10:00:00 // true
@2018-03-01T10 >= @2018-03-01T10:30 // empty ({ })
@2018-03-01T10:30:00 >= @2018-03-01T10:30:00.0 // true
@T10:30:00 >= @T10:00:00 // true
@T10 >= @T10:30 // empty ({ })
@T10:30:00 >= @T10:30:00.0 // true
6.3. Types
6.3.1. is type specifier
If the left operand is a collection with a single item and the second operand is a type identifier, this operator returns true if the type of the left operand is the type specified in the second operand, or a subclass thereof. If the identifier cannot be resolved to a valid type identifier, the evaluator will throw an error. If the input collections contains more than one item, the evaluator will throw an error. In all other cases this operator returns the empty collection.

A type specifier is an identifier that must resolve to the name of a type in a model. Type specifiers can have qualifiers, e.g. FHIR.Patient, where the qualifier is the name of the model.

Patient.contained.all($this is Patient implies age > 10)
This example returns true if for all the contained resources, if the contained resource is of type Patient, then the age is greater than ten.

6.3.2. is(type : TypeInfo)
The is() function is supported for backwards compatibility with previous implementations of FHIRPath. Just as with the is keyword, the type argument is an identifier that must resolve to the name of a type in a model. For implementations with compile-time typing, this requires special-case handling when processing the argument to treat is a type specifier rather than an identifier expression:

Patient.contained.all($this.is(Patient) implies age > 10)
Note: The is() function is defined for backwards compatibility only and may be deprecated in a future release.

6.3.3. as type specifier
If the left operand is a collection with a single item and the second operand is an identifier, this operator returns the value of the left operand if it is of the type specified in the second operand, or a subclass thereof. If the identifier cannot be resolved to a valid type identifier, the evaluator will throw an error. If there is more than one item in the input collection, the evaluator will throw an error. Otherwise, this operator returns the empty collection.

A type specifier is an identifier that must resolve to the name of a type in a model. Type specifiers can have qualifiers, e.g. FHIR.Patient, where the qualifier is the name of the model.

Observation.component.where(value as Quantity > 30 'mg')
6.3.4. as(type : TypeInfo)
The as() function is supported for backwards compatibility with previous implementations of FHIRPath. Just as with the as keyword, the type argument is an identifier that must resolve to the name of a type in a model. For implementations with compile-time typing, this requires special-case handling when processing the argument to treat is a type specifier rather than an identifier expression:

Observation.component.where(value.as(Quantity) > 30 'mg')
Note: The as() function is defined for backwards compatibility only and may be deprecated in a future release.

6.4. Collections
6.4.1. | (union collections)
Merge the two collections into a single collection, eliminating any duplicate values (using = (Equals) (=)) to determine equality). Unioning an empty collection to a non-empty collection will return the non-empty collection with duplicates eliminated. There is no expectation of order in the resulting collection.

6.4.2. in (membership)
If the left operand is a collection with a single item, this operator returns true if the item is in the right operand using equality semantics. If the left-hand side of the operator is empty, the result is empty, if the right-hand side is empty, the result is false. If the left operand has multiple items, an exception is thrown.

The following example returns true if 'Joe' is in the list of given names for the Patient:

'Joe' in Patient.name.given
6.4.3. contains (containership)
If the right operand is a collection with a single item, this operator returns true if the item is in the left operand using equality semantics. This is the converse operation of in.

The following example returns true if the list of given names for the Patient has 'Joe' in it:

Patient.name.given contains 'Joe'
6.5. Boolean logic
For all boolean operators, the collections passed as operands are first evaluated as Booleans (as described in Singleton Evaluation of Collections). The operators then use three-valued logic to propagate empty operands.

Note: To ensure that FHIRPath expressions can be freely rewritten by underlying implementations, there is no expectation that an implementation respect short-circuit evaluation. With regard to performance, implementations may use short-circuit evaluation to reduce computation, but authors should not rely on such behavior, and implementations must not change semantics with short-circuit evaluation. If short-circuit evaluation is needed to avoid effects (e.g. runtime exceptions), use the iff() function.

6.5.1. and
Returns true if both operands evaluate to true, false if either operand evaluates to false, and the empty collection ({ }) otherwise.

and	true	false	empty
true

true

false

empty ({ })

false

false

false

false

empty

empty ({ })

false

empty ({ })

6.5.2. or
Returns false if both operands evaluate to false, true if either operand evaluates to true, and empty ({ }) otherwise:

or	true	false	empty
true

true

true

true

false

true

false

empty ({ })

empty

true

empty ({ })

empty ({ })

6.5.3. not() : Boolean
Returns true if the input collection evaluates to false, and false if it evaluates to true. Otherwise, the result is empty ({ }):

not	
true

false

false

true

empty

empty ({ })

6.5.4. xor
Returns true if exactly one of the operands evaluates to true, false if either both operands evaluate to true or both operands evaluate to false, and the empty collection ({ }) otherwise:

xor	true	false	empty
true

false

true

empty ({ })

false

true

false

empty ({ })

empty

empty ({ })

empty ({ })

empty ({ })

6.5.5. implies
If the left operand evaluates to true, this operator returns the boolean evaluation of the right operand. If the left operand evaluates to false, this operator returns true. Otherwise, this operator returns true if the right operand evaluates to true, and the empty collection ({ }) otherwise.

implies	true	false	empty
true

true

false

empty ({ })

false

true

true

true

empty

true

empty ({ })

empty ({ })

The implies operator is useful for testing conditionals. For example, if a given name is present, then a family name must be as well:

Patient.name.given.exists() implies Patient.name.family.exists()
Note that implies may use short-circuit evaluation in the case that the first operand evaluates to false.

6.6. Math
The math operators require each operand to be a single element. Both operands must be of the same type, or of compatible types according to the rules for implicit conversion. Each operator below specifies which types are supported.

If there is more than one item, or an incompatible item, the evaluation of the expression will end and signal an error to the calling environment.

As with the other operators, the math operators will return an empty collection if one or both of the operands are empty.

When operating on quantities, the dimensions of each quantity must be the same, but not necessarily the unit. For example, units of 'cm' and 'm' can be compared, but units of 'cm2' and 'cm' cannot. The unit of the result will be the most granular unit of either input. Attempting to operate on quantities with invalid units will result in empty ({ }).

Implementations are not required to fully support operations on units, but they must at least respect units, recognizing when units differ.

Implementations that do support units SHALL do so as specified by [UCUM].

Operations that cause arithmetic overflow or underflow will result in empty ({ }).

6.6.1. * (multiplication)
Multiplies both arguments (supported for Integer, Decimal, and Quantity). For multiplication involving quantities, the resulting quantity will have the appropriate unit:

12 'cm' * 3 'cm' // 36 'cm2'
3 'cm' * 12 'cm2' // 36 'cm3'
6.6.2. / (division)
Divides the left operand by the right operand (supported for Integer, Decimal, and Quantity). The result of a division is always Decimal, even if the inputs are both Integer. For integer division, use the div operator.

If an attempt is made to divide by zero, the result is empty.

For division involving quantities, the resulting quantity will have the appropriate unit:

12 'cm2' / 3 'cm' // 4.0 'cm'
12 / 0 // empty ({ })
6.6.3. + (addition)
For Integer, Decimal, and quantity, adds the operands. For strings, concatenates the right operand to the left operand.

When adding quantities, the dimensions of each quantity must be the same, but not necessarily the unit.

3 'm' + 3 'cm' // 303 'cm'
6.6.4. - (subtraction)
Subtracts the right operand from the left operand (supported for Integer, Decimal, and Quantity).

When subtracting quantities, the dimensions of each quantity must be the same, but not necessarily the unit.

3 'm' - 3 'cm' // 297 'cm'
6.6.5. div
Performs truncated division of the left operand by the right operand (supported for Integer and Decimal). In other words, the division that ignores any remainder:

5 div 2 // 2
5.5 div 0.7 // 7
5 div 0 // empty ({ })
6.6.6. mod
Computes the remainder of the truncated division of its arguments (supported for Integer and Decimal).

5 mod 2 // 1
5.5 mod 0.7 // 0.6
5 mod 0 // empty ({ })
6.6.7. & (String concatenation)
For strings, will concatenate the strings, where an empty operand is taken to be the empty string. This differs from + on two strings, which will result in an empty collection when one of the operands is empty. This operator is specifically included to simplify treating an empty collection as an empty string, a common use case in string manipulation.

'ABC' + 'DEF' // 'ABCDEF'
'ABC' + { } + 'DEF' // { }
'ABC' & 'DEF' // 'ABCDEF'
'ABC' & { } & 'DEF' // 'ABCDEF'
6.7. Date/Time Arithmetic
Date and time arithmetic operators are used to add time-valued quantities to date/time values. The left operand must be a Date, DateTime, or Time value, and the right operand must be a Quantity with a time-valued unit:

year, years, or 'a'

month, months, or 'mo'

week, weeks or 'wk'

day, days, or 'd'

hour, hours, or 'h'

minute, minutes, or 'min'

second, seconds, or 's'

millisecond, milliseconds, or 'ms'

If there is more than one item, or an item of an incompatible type, the evaluation of the expression will end and signal an error to the calling environment.

If either or both arguments are empty ({ }), the result is empty ({ }).

6.7.1. + (addition)
Returns the value of the given Date, DateTime, or Time, incremented by the time-valued quantity, respecting variable length periods for calendar years and months.

For Date values, the quantity unit must be one of: years, months, weeks, or days

For DateTime values, the quantity unit must be one of: years, months, weeks, days, hours, minutes, seconds, or milliseconds (or an equivalent unit), or an error is raised.

For Time values, the quantity unit must be one of: hours, minutes, seconds, or milliseconds (or an equivalent unit), or an error is raised.

For partial date/time values, the operation is performed by converting the time-valued quantity to the highest precision in the partial (removing any decimal value off) and then adding to the date/time value. For example:

@2014 + 24 months
This expression will evaluate to the value @2016 even though the date/time value is not specified to the level of precision of the time-valued quantity.

Calculations involving weeks are equivalent to multiplying the number of weeks by 7 and performing the calculation for the resulting number of days.

Note: Although [UCUM] identifies 'a' as 365.25 days, and 'mo' as 1/12 of a year, calculations involving durations shall round using calendar semantics as specified in [ISO8601].

6.7.2. - (subtraction)
Returns the value of the given Date, DateTime, or Time, decremented by the time-valued quantity, respecting variable length periods for calendar years and months.

For Date values, the quantity unit must be one of: years, months, weeks, or days

For DateTime values, the quantity unit must be one of: years, months, weeks, days, hours, minutes, seconds, milliseconds (or an equivalent unit), or an error is raised.

For Time values, the quantity unit must be one of: hours, minutes, seconds, or milliseconds (or an equivalent unit), or an error is raised.

For partial date/time values, the operation is performed by converting the time-valued quantity to the highest precision in the partial (removing any decimal value off) and then subtracting from the date/time value. For example:

@2014 - 24 months
This expression will evaluate to the value @2012 even though the date/time value is not specified to the level of precision of the time-valued quantity.

Calculations involving weeks are equivalent to multiplying the number of weeks by 7 and performing the calculation for the resulting number of days.

Note: Although [UCUM] identifies 'a' as 365.25 days, and 'mo' as 1/12 of a year, calculations involving durations shall round using calendar semantics as specified in [ISO8601].

6.8. Operator precedence
Precedence of operations, in order from high to low:

#01 . (path/function invocation)
#02 [] (indexer)
#03 unary + and -
#04: *, /, div, mod
#05: +, -, &
#06: is, as
#07: |
#08: >, <, >=, <=
#09: =, ~, !=, !~
#10: in, contains
#11: and
#12: xor, or
#13: implies
As customary, expressions may be grouped by parenthesis (()).

7. Aggregates
Note: the contents of this section are Standard for Trial Use (STU)

FHIRPath supports a general-purpose aggregate function to enable the calculation of aggregates such as sum, min, and max to be expressed:

7.1. aggregate(aggregator : expression [, init : value]) : value
Performs general-purpose aggregation by evaluating the aggregator expression for each element of the input collection. Within this expression, the standard iteration variables of $this and $index can be accessed, but also a $total aggregation variable.

The value of the $total variable is set to init, or empty ({ }) if no init value is supplied, and is set to the result of the aggregator expression after every iteration.

Using this function, sum can be expressed as:

value.aggregate($this + $total, 0)
Min can be expressed as:

value.aggregate(iif($total.empty(), $this, iif($this < $total, $this, $total)))
and average would be expressed as:

value.aggregate($total + $this, 0) / value.count()
8. Lexical Elements
FHIRPath defines the following lexical elements:

Element	Description
Whitespace

Whitespace defines the separation between tokens in the language

Comment

Comments are ignored by the language, allowing for descriptive text

Literal

Literals allow basic values to be represented within the language

Symbol

Symbols such as +, -, *, and /

Keyword

Grammar-recognized tokens such as and, or and in

Identifier

Labels such as type names and property names

8.1. Whitespace
FHIRPath defines tab (\t), space ( ), line feed (\n) and carriage return (\r) as whitespace, meaning they are only used to separate other tokens within the language. Any number of whitespace characters can appear, and the language does not use whitespace for anything other than delimiting tokens.

8.2. Comments
FHIRPath defines two styles of comments, single-line, and multi-line. A single-line comment consists of two forward slashes, followed by any text up to the end of the line:

2 + 2 // This is a single-line comment
To begin a multi-line comment, the typical forward slash-asterisk token is used. The comment is closed with an asterisk-forward slash, and everything enclosed is ignored:

/*
This is a multi-line comment
Any text enclosed within is ignored
*/
8.3. Literals
Literals provide for the representation of values within FHIRPath. The following types of literals are supported:

Literal	Description
Empty ({ })

The empty collection

Boolean

The boolean literals (true and false)

Integer

Sequences of digits in the range 0..232-1

Decimal

Sequences of digits with a decimal point, in the range (-1028+1)/108..(1028-1)/108

String

Strings of any character enclosed within single-ticks (')

Date

The at-symbol (@) followed by a date (YYYY-MM-DD)

DateTime

The at-symbol (@) followed by a datetime (YYYY-MM-DDThh:mm:ss.fff(+/-)hh:mm)

Time

The at-symbol (@) followed by a time (Thh:mm:ss.fff(+/-)hh:mm)

Quantity

An integer or decimal literal followed by a datetime precision specifier, or a [UCUM] unit specifier

For a more detailed discussion of the semantics of each type, refer to the link for each type.

8.4. Symbols
Symbols provide structure to the language and allow symbolic invocation of common operators such as addition. FHIRPath defines the following symbols:

Symbol	Description
()

Parentheses for delimiting groups within expressions

[]

Brackets for indexing into lists and strings

{}

Braces for delimiting lists

.

Period for qualifiers, accessors, and dot-invocation

,

Comma for delimiting items in a syntactic list

= != <= < > >=

Comparison operators for comparing values

+ - * / | &

Arithmetic and other operators for performing computation

8.5. Keywords
Keywords are tokens that are recognized by the parser and used to build the various language constructs. FHIRPath defines the following keywords:

$index

div

milliseconds

true

$this

false

minute

week

$total

hour

minutes

weeks

and

hours

mod

xor

as

implies

month

year

contains

in

months

years

day

is

or

second

days

millisecond

seconds

In general, keywords within FHIRPath are also considered reserved words, meaning that it is illegal to use them as identifiers. If necessary, identifiers that clash with a reserved word can be delimited using a backtick ( ` ).

8.6. Identifiers
Identifiers are used as labels to allow expressions to reference elements such as model types and properties. FHIRPath supports two types of identifiers, simple and delimited.

A simple identifier is any alphabetical character or an underscore, followed by any number of alpha-numeric characters or underscores. For example, the following are all valid simple identifiers:

Patient
_id
valueDateTime
_1234
A delimited identifier is any sequence of characters enclosed in backticks ( ` ):

`QI-Core Patient`
`US-Core Diagnostic Request`
`us-zip`
The use of backticks allows identifiers to contains spaces, commas, and other characters that would not be allowed within simple identifiers. This allows identifiers to be more descriptive, and also enables expressions to reference models that have property or type names that are not valid simple identifiers.

FHIRPath escape sequences for strings also work for delimited identifiers.

When resolving an identifier that is also the root of a FHIRPath expression, it is resolved as a type name first, and if it resolves to a type, it must resolve to the type of the context (or a supertype). Otherwise, it is resolved as a path on the context. If the identifier cannot be resolved, an error is raised.

8.7. Case-Sensitivity
FHIRPath is a case-sensitive language, meaning that case is considered when matching keywords in the language. However, because FHIRPath can be used with different models, the case-sensitivity of type and property names is defined by each model.

9. Environment variables
A token introduced by a % refers to a value that is passed into the evaluation engine by the calling environment. Using environment variables, authors can avoid repetition of fixed values and can pass in external values and data.

The following environmental values are set for all contexts:

%ucum       // (string) url for UCUM (http://unitsofmeasure.org, per http://hl7.org/fhir/ucum.html)
%context    // The original node that was passed to the evaluation engine before starting evaluation
Implementers should note that using additional environment variables is a formal extension point for the language. Various usages of FHIRPath may define their own externals, and implementers should provide some appropriate configuration framework to allow these constants to be provided to the evaluation engine at run-time. E.g.:

%`us-zip` = '[0-9]{5}(-[0-9]{4}){0,1}'
Note that the identifier portion of the token is allowed to be either a simple identifier (as in %ucum), or a delimited identifier to allow for alternative characters (as in %`us-zip`).

Note also that these tokens are not restricted to simple types, and they may have values that are not defined fixed values known prior to evaluation at run-time, though there is no way to define these kind of values in implementation guides.

Attempting to access an undefined environment variable will result in an error, but accessing a defined environment variable that does not have a value specified results in empty ({ }).

Note: For backwards compatibility with some existing implementations, the token for an environment variable may also be a string, as in %'us-zip'.

10. Types and Reflection
10.1. Models
Because FHIRPath is defined to work in multiple contexts, each context provides the definition for the structures available in that context. These structures are the model available for FHIRPath expressions. For example, within FHIR, the FHIR data types and resources are the model. To prevent namespace clashes, the type names within each model are prefixed (or namespaced) with the name of the model. For example, the fully qualified name of the Patient resource in FHIR is FHIR.Patient. The system types defined within FHIRPath directly are prefixed with the namespace System.

To allow type names to be referenced in expressions such as the is and as operators, the language includes a type specifier, an optionally qualified identifier that must resolve to the name of a model type.

When resolving a type name, the context-specific model is searched first. If no match is found, the System model (containing only the built-in types defined in the Literals section) is searched.

When resolving an identifier that is also the root of a FHIRPath expression, it is resolved as a type name first, and if it resolves to a type, it must resolve to the type of the context (or a supertype). Otherwise, it is resolved as a path on the context.

10.2. Reflection
Note: The contents of this section are Standard for Trial Use (STU)

FHIRPath supports reflection to provide the ability for expressions to access type information describing the structure of values. The type() function returns the type information for each element of the input collection, using one of the following concrete subtypes of TypeInfo:

10.2.1. Primitive Types
For primitive types such as String and Integer, the result is a SimpleTypeInfo:

SimpleTypeInfo { namespace: string, name: string, baseType: TypeSpecifier }
For example:

('John' | 'Mary').type()
Results in:

{
  SimpleTypeInfo { namespace: 'System', name: 'String', baseType: 'System.Any' },
  SimpleTypeInfo { namespace: 'System', name: 'String', baseType: 'System.Any' }
}
10.2.2. Class Types
For class types, the result is a ClassInfo:

ClassInfoElement { name: string, type: TypeSpecifier, isOneBased: Boolean }
ClassInfo { namespace: string, name: string, baseType: TypeSpecifier, element: List<ClassInfoElement> }
For example:

Patient.maritalStatus.type()
Results in:

{
  ClassInfo {
    namespace: 'FHIR',
    name: 'CodeableConcept',
    baseType: 'FHIR.Element',
    element: {
      ClassInfoElement { name: 'coding', type: 'List<Coding>', isOneBased: false },
      ClassInfoElement { name: text', type: 'FHIR.string' }
    }
  }
}
10.2.3. Collection Types
For collection types, the result is a ListTypeInfo:

ListTypeInfo { elementType: TypeSpecifier }
For example:

Patient.address.type()
Results in:

{
  ListTypeInfo { elementType: 'FHIR.Address' }
}
10.2.4. Anonymous Types
And for anonymous types, the result is a TupleTypeInfo:

TupleTypeInfoElement { name: string, type: TypeSpecifier, isOneBased: Boolean }
TupleTypeInfo { element: List<TupleTypeInfoElement> }
For example:

Patient.contact.type()
Results in:

{
  TupleTypeInfo {
    element: {
      TupleTypeInfoElement { name: 'relationship', type: 'List<FHIR.CodeableConcept>', isOneBased: false },
      TupleTypeInfoElement { name: 'name', type: 'FHIR.HumanName', isOneBased: false },
      TupleTypeInfoElement { name: 'telecom', type: 'List<FHIR.ContactPoint>', isOneBased: false },
      TupleTypeInfoElement { name: 'address', type: 'FHIR.Address', isOneBased: false },
      TupleTypeInfoElement { name: 'gender', type: 'FHIR.code', isOneBased: false },
      TupleTypeInfoElement { name: 'organization', type: 'FHIR.Reference', isOneBased: false },
      TupleTypeInfoElement { name: 'period', type: 'FHIR.Period', isOneBased: false }
    }
  }
}
Note: These structures are a subset of the abstract metamodel used by the Clinical Quality Language Tooling.

11. Type safety and strict evaluation
Strongly typed languages are intended to help authors avoid mistakes by ensuring that the expressions describe meaningful operations. For example, a strongly typed language would typically disallow the expression:

1 + 'John'
because it performs an invalid operation, namely adding numbers and strings. However, there are cases where the author knows that a particular invocation may be safe, but the compiler is not aware of, or cannot infer, the reason. In these cases, type-safety errors can become an unwelcome burden, especially for experienced developers.

Because FHIRPath may be used in different situations and environments requiring different levels of type safety, implementations may make different choices about how much type checking should be done at compile-time versus run-time, and in what situations. Some implementations requiring a high degree of type-safety may choose to perform strict type-checking at compile-time for all invocations. On the other hand, some implementations may be unconcerned with compile-time versus run-time checking and may choose to defer all correctness checks to run-time.

For example, since some functions and most operators will only accept a single item as input (and throw a run-time exception otherwise):

Patient.name.given + ' ' + Patient.name.family
will work perfectly fine, as long as the patient has a single name, but will fail otherwise. It is in fact "safer" to formulate such statements as either:

Patient.name.select(given + ' ' + family)
which would return a collection of concatenated first and last names, one for each name of a patient. Of course, if the patient turns out to have multiple given names, even this statement will fail and the author would need to choose the first name in each collection explicitly:

Patient.name.first().select(given.first() + ' ' + family.first())
It is clear that, although more robust, the last expression is also much more elaborate, certainly in situations where, because of external constraints, the author is sure names will not repeat, even if the unconstrained data model allows repetition.

Apart from throwing exceptions, unexpected outcomes may result because of the way the equality operators are defined. The expression

Patient.name.given = 'Wouter'
will return false as soon as a patient has multiple names, even though one of those may well be 'Wouter'. Again, this can be corrected:

Patient.name.where(given = 'Wouter').exists()
but is still less concise than would be possible if constraints were well known in advance.

In cases where compile-time checking like this is desirable, implementations may choose to protect against such cases by employing strict typing. Based on the definitions of the operators and functions involved in the expression, and given the types of the inputs, a compiler can analyze the expression and determine whether "unsafe" situations can occur.

Unsafe uses are:

A function that requires an input collection with a single item is called on an output that is not guaranteed to have only one item.

A function is passed an argument that is not guaranteed to be a single value.

A function is passed an input value or argument that is not of the expected type

An operator that requires operands to be collections with a single item is called with arguments that are not guaranteed to have only one item.

An operator has operands that are not of the expected type

Equality operators are used on operands that are not both collections or collections containing a single item of the same type.

There are a few constructs in the FHIRPath language where the compiler cannot determine the type:

The children() and descendants() functions

The resolve() function

A member which is polymorphic (e.g. a choice[x] type in FHIR)

Note that the resolve() function is defined by the FHIR context, it is not part of FHIRPath directly. For more information see the FHIRPath section of the FHIR specification.

Authors can use the as operator or ofType() function directly after such constructs to inform the compiler of the expected type.

In cases where a compiler finds places where a collection of multiple items can be present while just a single item is expected, the author will need to make explicit how repetitions are dealt with. Depending on the situation one may:

Use first(), last() or indexer ([ ]) to select a single item

Use select() and where() to turn the expression into one that evaluates each of the repeating items individually (as in the examples above)

12. Formal Specifications
12.1. Formal Syntax
The formal syntax for FHIRPath is specified as an Antlr 4.0 grammar file (g4) and included in this specification at the following link:

grammar.html

12.2. Model Information
The model information returned by the reflection function type() is specified as an XML Schema document (xsd) and included in this specification at the following link:

modelinfo.xsd

Note: The model information file included here is not a normative aspect of the FHIRPath specification. It is the same model information file used by the Clinical Quality Framework Tooling and is included for reference as a simple formalism that meets the requirements described in the normative Reflection section above.

As discussed in the section on case-sensitivity, each model used within FHIRPath determines whether or not identifiers in the model are case-sensitive. This information is provided as part of the model information and tooling should respect the case-sensitive settings for each model.

12.3. URI and Media Types
To uniquely identify the FHIRPath language, the following URI is defined:

http://hl7.org/fhirpath

In addition, a media type is defined to support describing FHIRPath content:

text/fhirpath

Note: The appendices are included for informative purposes and are not a normative part of the specification.

Appendix A: Use of FHIRPath on HL7 Version 2 messages
FHIRPath can be used against HL7 V2 messages. This UML diagram summarises the object model on which the FHIRPath statements are written:

Class Model for HL7 V2

In this Object Model:

The object graph always starts with a message.

Each message has a list of segments.

In addition, Abstract Message Syntax is available through the groups() function, for use where the message follows the Abstract Message Syntax sufficiently for the parser to reconcile the segment list with the structure.

The names of the groups are the names published in the specification, e.g. 'PATIENT_OBSERVATION' (with spaces, where present, replaced by underscores. In case of doubt, consult the V2 XML schemas).

Each Segment has a list of fields, which each have a list of "Cells". This is necessary to allow for repeats, but users are accustomed to just jumping to Element - use the function elements() which returns all repeats with the given index.

A "cell" can be either an Element, a Component or a Sub-Components. Elements can contain Components, which can contain Sub-Components. Sub-Sub-Components are not allowed.

Calls may have a simple text content, or a series of (sub-)components. The simple() function returns either the text, if it exists, or the return value of simple() from the first component

A V2 data type (e.g. ST, SN, CE etc) is a profile on Cell that specifies whether it has simple content, or complex content.

todo: this object model doesn’t make provision for non-syntax escapes in the simple content (e.g. \.b\).

all the lists are 1 based. That means the first item in the list is numbered 1, not 0.

Some example queries:

Message.segment.where(code = 'PID').field[3].element.first().simple()
Get the value of the first component in the first repeat of PID-3

Message.segment[2].elements(3).simple()
Get a collection with is the string values of all the repeats in the the 3rd element of the 2nd segement. Typically, this assumes that there is no repeats, and so this is a simple value

Message.segment.where(code = 'PID').field[3].element.where(component[4].value = 'MR').simple()
Pick out the MR number from PID-3 (assuming, in this case, that there’s only one PID segment in the message. No good for an A17). Note that this returns the whole Cell - e.g. |value^^MR|, though often more components will be present)

Message.segment.where(code = 'PID').elements(3).where(component[4].value = 'MR').component[1].text
Same as the last, but pick out just the MR value

Message.group('PATIENT').group('PATIENT_OBSERVATION').item.ofType(Segment)
  .where(code = 'OBX' and elements(2).exists(components(2) = 'LN')))
return any OBXs from the patient observations (and ignore others e.g. in a R01 message) segments that have LOINC codes. Note that if the parser cannot properly parse the Abstract Message Syntax, group() must fail with an error message.

Appendix B: FHIRPath Tooling and Implementation
This section lists known tooling and implementation projects for the FHIRPath language:

JavaScript: http://niquola.github.io/fhirpath-demo/#/

Java RI: In the FHIR build tooling at org.hl7.fhir.dstu3.utils.FHIRPathEngine

Pascal RI: https://github.com/grahamegrieve/fhirserver/blob/master/reference-platform/dstu3/FhirPath.pas

.NET RI: https://github.com/ewoutkramer/fhir-net-fhirpath

In addition, there is a Notepad++ FHIR Plugin that enables evaluation of FHIRPath expressions:

http://www.healthintersections.com.au/?p=2386

There is a test harness for FHIRPath here:

https://github.com/brianpos/FhirPathTester

The CQL-to-ELM translator that is maintained as part of the tooling for Clinical Quality Language supports FHIRPath:

https://github.com/cqframework/clinical_quality_language

For the most current listing of known implementations, refer to the HL7 wiki:

http://wiki.hl7.org/index.php?title=FHIRPath_Implementations

Appendix C: References
[ANTLR] Another Tool for Language Recognition (ANTLR) http://www.antlr.org/

[ISO8601] Date and time format - ISO 8601. https://www.iso.org/iso-8601-date-and-time-format.html

[CQL] HL7 Cross-Paradigm Specification: Clinical Quality Language, Release 1, STU Release 1.3. http://www.hl7.org/implement/standards/product_brief.cfm?product_id=400

[MOF] Meta Object Facility. https://www.omg.org/spec/MOF/, version 2.5.1, November 2016

[XMLRE] Regular Expressions. XML Schema 1.1. https://www.w3.org/TR/xmlschema11-2/#regexs

[PCRE] Pearl-Compatible Regular Expressions. http://www.pcre.org/

[UCUM] Unified Code for Units of Measure (UCUM) http://unitsofmeasure.org/ucum.html, Version 2.1, Revision 442 (2017-11-21)
