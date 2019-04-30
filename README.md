FHIRPath (STU1 Release)
FHIRPath is a path based navigation and extraction language, somewhat like XPath. Operations are expressed in terms of the logical content of hierarchical data models, and support traversal, selection and filtering of data. Its design was influenced by the needs for path navigation, selection and formulation of invariants in both HL7 Fast Healthcare Interoperability Resources (FHIR) and HL7 Clinical Quality Language (CQL).

Looking for implementations? See FHIRPath Implementations on the HL7 wiki

Version: 1.0.0 Public Domain (Creative Commons 0)

Table of Contents
1. Overview
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
4.3. Functions
4.4. Null and empty
4.5. Singleton evaluation of collections
5. Functions
5.1. Existence
5.2. Filtering and projection
5.3. Subsetting
5.4. Combining
5.5. Conversion
5.6. String Manipulation
5.7. Tree navigation
5.8. Utility functions
6. Operations
6.1. Equality
6.2. Comparison
6.3. Types
6.4. Collections
6.5. Boolean logic
6.6. Math
6.7. Date/Time Arithmetic
6.8. Operator precedence
7. Lexical Elements
7.1. Whitespace
7.2. Comments
7.3. Literals
7.4. Symbols
7.5. Keywords
7.6. Identifiers
7.7. Case-Sensitivity
8. Environment variables
9. Reflection
10. Type safety and strict evaluation
Appendix A: Formal Specifications
A.1. Formal Syntax
A.2. Model Information
A.3. URI and Media Types
Appendix B: Use of FHIRPath in HL7 FHIR
B.1. Polymorphism in FHIR
B.2. Using FHIR types in expressions
B.3. Additional functions
B.4. Changes to operators
B.5. Environment variables
Appendix C: Use of FHIRPath in Clinical Quality Language (CQL)
C.1. Path Traversal
C.2. Constants and Contexts
C.3. Additional Operators
C.4. Method-style Invocation
C.5. Strict Evaluation
Appendix D: Use of FHIRPath on HL7 Version 2 messages
Appendix E: References
1. Overview
In Information Systems in general, and Healthcare Information Systems in particular, the need for formal representation of logic is both pervasive and critical. From low-level technical specifications, through intermediate logical architectures, up to the high-level conceptual descriptions of requirements and behavior, the ability to formally represent knowledge in terms of expressions and information models is essential to the specification and implementation of these systems.

1.1. Requirements
Of particular importance is the ability to easily and precisely express conditions of basic logic, such as those found in requirements constraints (e.g. Patients must have a name), decision support (e.g. if the patient has diabetes and has not had a recent comprehensive foot exam), cohort definitions (e.g. All male patients aged 60-75), protocol descriptions (e.g. if the specimen has tested positive for the presence of sodium), and numerous other environments.

Precisely because the need for such expressions is so pervasive, there is no shortage of existing languages for representing them. However, these languages tend to be tightly coupled to the data structures, and even the information models on which they operate, XPath being a typical example. To ensure that the knowledge captured by the representation of these expressions can survive technological drift, a representation that can be used independent of any underlying physical implementation and information model is required.

Languages meeting these additional requirements also exist, such as Java, JavaScript, C#, and others. However, these languages are both tightly coupled to the platforms in which they operate, and, because they are general-purpose development languages, come with much heavier tooling and technology dependencies than is warranted or desirable. Even constraining one of these grammars would be insufficient, resulting in the need to extend, defeating the purpose of basing it on an existing language in the first place.

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

FHIRPath can be used against many other graphs as well. For example, Use of FHIRPath on HL7 Version 2 messages describes how FHIRPath is used in HL7 v2.

1.4. Conventions
Throughout this documentation, monospace font is used to delineate expressions of FHIRPath.

Optional parameters to functions are enclosed in square brackets in the definition of a function. Note that the brackets are only used to indicate optionality in the signature, they are not part of the actual syntax of FHIRPath.

All functions return a collection, but if the function or operation will always produce a collection containing a single item of a predefined type, the description of the function will specify its output type explicitly, instead of just stating collection, e.g. all(…​) : boolean

2. Navigation model
FHIRPath navigates and selects nodes from a tree that abstracts away and is independent of the actual underlying implementation of the source against which the FHIRPath query is run. This way, FHIRPath can be used on in-memory Java POJOs, Xml data or any other physical representation, so long as that representation can be viewed as classes that have properties. In somewhat more formal terms, FHIRPath operates on a directed acyclic graph of classes as defined by a MOF-equivalent type system.

Data are represented as a tree of labelled nodes, where each node may optionally carry a primitive value and have child nodes. Nodes need not have a unique label, and leaf nodes must carry a primitive value. For example, a (partial) representation of a FHIR Patient resource in this model looks like this:

Tree representation of a Patient

The diagram shows a tree with a repeating name node, which represents repeating members of the FHIR object model. Leaf nodes such as use and family carry a (string) value. It is also possible for internal nodes to carry a value, as is the case for the node labelled active: this allows the tree to represent FHIR "primitives", which may still have child extension data.

3. Path selection
FHIRPath allows navigation through the tree by composing a path of concatenated labels, e.g.

----name.given
----
This would result in a collection of nodes, one with the value "Wouter" and one with the value "Gert". In fact, each step in such a path results in a collection of nodes by selecting nodes with the given label from the step before it. The focus at the beginning of the evaluation contained all elements from Patient, and the path name selected just those named name. Since the name element repeats, the next step given along the path, will contain all nodes labeled given from all nodes name in the preceding step.

The path may start with the type of the root node (which otherwise does not have a name), but this is optional. To illustrate this point, the path name.given above can be evaluated as an expression on a set of data of any type. However the expression may be prefixed with the name of the type of the root:

Patient.name.given
The two expressions have the same outcome, but when evaluating the second, the evaluation will only produce results when used on data of type Patient.

Syntactically, FHIRPath defines identifiers as any sequence of characters consisting only of letters, digits, and underscores, beginning with a letter or underscore. Paths may use double quotes to include characters in path parts that would otherwise be interpreted as keywords or operators, e.g.:

Message."PID-1"
3.1. Collections
Collections are fundamental to FHIRPath, in that the result of every expression is a collection, even if that expression only results in a single element. This approach allows paths to be specified without having to care about the cardinality of any particular element, and is therefore ideally suited to graph traversal.

Within FHIRPath, a collection is:

Ordered - The order of items in the collection is important and is preserved through operations as much as possible.

Non-Unique - Duplicate elements are allowed within a collection. Some functions, such as distinct() and the union operator | produce collections of unique elements, but in general, duplicate elements are allowed.

Indexed - Each item in a collection can be uniquely addressed by it’s index, i.e. ordinal position within the collection. The first item in a collection has index 0.

Countable - The number of items in a given collection can always be determined using the count() function

Note that the outcome of operations like children() and descendants() cannot be assumed to be in any meaningful order, and first(), last(), tail(), skip() and take() should not be used on collections derived from these paths. Note that some implementations may follow the logical order implied by the data model, and some may not, and some may be different depending on the underlying source.

3.2. Paths and polymorphic items
In the underlying representation of data, nodes may be typed and represent polymorphic items. Paths may either ignore the type of a node, and continue along the path or may be explicit about the expected node and filter the set of nodes by type before navigating down child nodes:

Observation.value.unit - all kinds of value
Observation.value.ofType(Quantity).unit - only values that are of type Quantity
The is operator can be used to determine whether or not a given value is of a given type:

Observation.value is Quantity - returns true if the value is of type Quantity
The as operator can be used to treat a value as a specific type:

Observation.value as Quantity - returns value as a Quantity if it is of type Quantity, and an empty result otherwise
The list of available types that can be passed as a parameter to the ofType() function and is and as operators is determined by the underlying data model. Within FHIRPath, they are just identifiers, either quoted or non-quoted.

4. Expressions
4.1. Literals
In addition to paths, FHIRPath expressions may contain literals and function invocations. FHIRPath supports the following types of literals:

boolean: true, false
string: 'test string', 'urn:oid:3.4.5.6.7.8'
integer: 0, 45
decimal: 0.0, 3.141592653589793236
dateTime: @2015-02-04T14:34:28Z (`@` followed by ISO8601 compliant date/time)
time: @T14:34:28+09:00 (`@` followed by ISO8601 compliant time beginning with `T`)
quantity: 10 'mg', 4 days
4.1.1. string
Unicode is supported in both string literals and quoted identifiers. String literals are surrounded by single quotes and may use \-escapes to escape quotes and represent Unicode characters:

Unicode characters may be escaped using \u followed by four hex digits.

Additional escapes are those supported in JSON:

\\ (backslash),

\/ (slash),

\f (form feed - \u000c),

\n (newline - \u000a),

\r (carriage return - \u000d),

\t (tab - \u0009)

\" (double quote)

\' (single quote)

4.1.2. decimal
Decimals cannot use exponential notation.

4.1.3. dateTime
dateTime uses a subset of ISO8601:

It uses the YYYY-MM-DD format, though month and day parts are optional

Week dates and ordinal dates are not allowed

Years must be present (-MM-DD is not a valid dateTime in FHIRPath)

Months must be present if a day is present

The date may be followed by a time as described in the next section.

Consult the formal grammar for more details.

4.1.4. time
time uses a subset of ISO8601:

A time begins with a T

Timezone is optional, but if present the notation ±hh:mm is used (so must include both minutes and hours)

Z is allowed as a synonym for the zero (+00:00) UTC offset.

Consult the formal grammar for more details.

4.1.5. quantity
quantity is a number (integer or decimal), followed by a (single quoted) string representing a valid Unified Code for Units of Measure (UCUM) unit:

  4.5 'mg'
  100 '[degF]'
For date/time units, an alternative representation may be used (note that both a plural and singular version exist):

year/years, month/months, week/weeks, day/days, hour/hours, minute/minutes, second/seconds, millisecond/milliseconds

  1 year
  4 days
4.2. Operators
Expressions can also contain operators, like those for mathematical operations and boolean logic:

Appointment.minutesDuration / 60 > 5
MedicationAdministration.wasNotGiven implies MedicationAdministration.reasonNotGiven.exists()
name.given | name.family // union of given and family names
'sir ' + name.given
4.3. Functions
Finally, FHIRPath supports the notion of functions, which all take a collection of values as input and produce another collection as output and may take parameters. For example:

(name.given | name.family).substring(0,4)
identifier.where(use = 'official')
Since all functions work on collections, constants will first be converted to a collection when functions are invoked on constants:

(4+5).count()
will return 1, since this is implicitly a collection with one constant number 9.

4.4. Null and empty
There is no concept of null in FHIRPath. This means that when, in an underlying data object a member is null or missing, there will simply be no corresponding node for that member in the tree, e.g. Patient.name will return an empty collection (not null) if there are no name elements in the instance.

In expressions, the empty collection is represented as {}.

4.4.1. Propagation of empty results in expressions
FHIRPath functions and operators both propagate empty results, but the behavior is in general different when the argument to the function or operator expects a collection (e.g. select(), where() and | (union)) versus when the argument to the function or operator takes a single value as input (e.g. + and substring()).

For functions or operators that take a single values as input, this means in general if the input is empty, then the result will be empty as well. More specifically:

If a single-input function operates on an empty collection, the result is an empty collection

If a single-input function is passed an empty collection as an argument, the result is an empty collection

If any operand to a single-input operator is an empty collection, the result is an empty collection.

For functions or arguments that expect collections, in general the empty collection is treated as any other collection would be. For example, the union (|) of an empty collection with a non-empty collection is the non-empty collection.

When functions or operators behave differently from these general principles, (for example the count() and empty() functions), this is clearly documented in the next sections.

4.5. Singleton evaluation of collections
In general, when a collection is passed as an argument to a function or operator that expects a single item as input, the collection is implicitly converted to a singleton as follows:

IF the collection contains a single node AND the node's value is of the expected input type THEN
  The collection evaluates to the value of that single node
ELSE IF the collection is empty THEN
  The collection evaluates to an empty collection
ELSE
  An error is raised
5. Functions
Functions are distinguished from path navigation names by the fact that they are followed by a () with zero or more parameters. With a few minor exceptions (e.g. the today() function), functions in FHIRPath always take a collection as input and produce another collection as output, even though these may be collections of just a single item.

Correspondingly, arguments to the functions can be any FHIRPath expression, though functions taking a single item as input require these expressions to evaluate to a collection containing a single item of a specific type. This approach allows functions to be chained, successively operating on the results of the previous function in order to produce the desired final result.

The following sections describe the functions supported in FHIRPath, detailing the expected types of parameters and type of collection returned by the function:

If the function expects a parameter to be a single value (e.g. item(index: integer) and it is passed an argument that evaluates to a collection with multiple items or a collection with an item that is not of the required type, the evaluation of the expression will end and an error will be signaled to the calling environment.

If the function takes an expression as a parameter, the function will evaluate this parameter with respect to each of the items in the input collection. These expressions may refer to the special $this element, which represents the item from the input collection currently under evaluation. For example, in name.given.where($this > 'ba' and $this < 'bc') the where() function will iterate over each item in the input collection (elements named given) and $this will be set to each item when the expression passed to where() is evaluated.

Note that the bracket notation in function signatures indicates optional parameters, and is not part of the formal syntax of FHIRPath.

Note also that although all functions return collections, if a given function is defined to return a single element function, the return type is simplified to just the type of the single element, rather than the list type.

5.1. Existence
5.1.1. empty() : boolean
Returns true if the input collection is empty ({ }) and false otherwise.

5.1.2. not() : boolean
Returns true if the input collection evaluates to false, and false if it evaluates to true. Otherwise, the result is empty ({ }):

 	not
true

false

false

true

empty ({ })

empty ({ })

5.1.3. exists([criteria : expression]) : boolean
Returns true if the collection has any elements, and false otherwise. This is the opposite of empty(), and as such is a shorthand for empty().not(). If the input collection is empty ({ }), the result is false.

The operator can also take an optional criteria to be applied to the collection prior to the determination of the exists. In this case, the operation is shorthand for where(criteria).exists().

5.1.4. all(criteria : expression) : boolean
Returns true if for every element in the input collection, criteria evaluates to true. Otherwise, the result is false. If the input collection is empty ({ }), the result is true.

5.1.5. allTrue() : boolean
Takes a collection of boolean values and returns true if all the items are true. If any items are false, the result is false. If the input is empty ({ }), the result is true.

5.1.6. anyTrue() : boolean
Takes a collection of boolean values and returns true if any of the items are true. If all the items are false, or if the input is empty ({ }), the result is false.

5.1.7. allFalse() : boolean
Takes a collection of boolean values and returns true if all the items are false. If any items are true, the result is false. If the input is empty ({ }), the result is true.

5.1.8. anyFalse() : boolean
Takes a collection of boolean values and returns true if any of the items are false. If all the items are true, or if the input is empty ({ }), the result is false.

5.1.9. subsetOf(other : collection) : boolean
Returns true if all items in the input collection are members of the collection passed as the other argument. Membership is determined using the equals (=) operation (see below).

Conceptually, this function is evaluated by testing each element in the input collection for membership in the other collection, with a default of true. This means that if the input collection is empty ({ }), the result is true, otherwise if the other collection is empty ({ }), the result is false.

5.1.10. supersetOf(other : collection) : boolean
Returns true if all items in the collection passed as the other argument are members of the input collection. Membership is determined using the equals (=) operation (see below).

Conceptually, this function is evaluated by testing each element in the other collection for membership in the input collection, with a default of false. This means that if the input collection is empty ({ }), the result is false, otherwise if the other collection is empty ({ }), the result is true.

5.1.11. isDistinct() : boolean
Returns true if all the items in the input collection are distinct. To determine whether two items are distinct, the equals (=) operator is used, as defined below.

Conceptually, this function is shorthand for a comparison of the count() of the input collection against the count() of the distinct() of the input collection:

X.count() = X.distinct().count()
This means that if the input collection is empty ({ }), the result is true.

5.1.12. distinct() : collection
Returns a collection containing only the unique items in the input collection. To determine whether two items are the same, the equals (=) operator is used, as defined below.

If the input collection is empty ({ }), the result is empty.

5.1.13. count() : integer
Returns a collection with a single value which is the integer count of the number of items in the input collection. Returns 0 when the input collection is empty.

5.2. Filtering and projection
5.2.1. where(criteria : expression) : collection
Returns a collection containing only those elements in the input collection for which the stated criteria expression evaluates to true. Elements for which the expression evaluates to false or empty ({ }) are not included in the result.

If the input collection is emtpy ({ }), the result is empty.

5.2.2. select(projection: expression) : collection
Evaluates the projection expression for each item in the input collection. The result of each evaluation is added to the output collection. If the evaluation results in a collection with multiple items, all items are added to the output collection (collections resulting from evaluation of projection are flattened). This means that if the evaluation for an element results in the empty collection ({ }), no element is added to the result, and that if the input collection is empty ({ }), the result is empty as well.

5.2.3. repeat(projection: expression) : collection
A version of select that will repeat the projection and add it to the output collection, as long as the projection yields new items (as determined by the equals (=) operator).

This operation can be used to traverse a tree and selecting only specific children:

ValueSet.expansion.repeat(contains)
Will repeat finding children called contains, until no new nodes are found.

Questionnaire.repeat(group | question).question
Will repeat finding children called group or question, until no new nodes are found.

Note that this is slightly different from:

Questionnaire.descendants().select(group | question)
which would find any descendants called group or question, not just the ones nested inside other group or question elements.

5.2.4. ofType(type : identifier) : collection
Returns a collection that contains all items in the input collection that are of the given type or a subclass thereof. If the input collection is empty ({ }), the result is empty.

5.3. Subsetting
5.3.1. [ index : integer ] : collection
The indexer operation returns a collection with only the index-th item (0-based index). If the input collection is empty ({ }), or the index lies outside the boundaries of the input collection, an empty collection is returned.

Example:

Patient.name[0]
5.3.2. single() : collection
Will return the single item in the input if there is just one item. If the input collection is empty ({ }), the result is empty. If there are multiple items, an error is signaled to the evaluation environment. This operation is useful for ensuring that an error is returned if an assumption about cardinality is violated at run-time.

5.3.3. first() : collection
Returns a collection containing only the first item in the input collection. This function is equivalent to item(0), so it will return an empty collection if the input collection has no items.

5.3.4. last() : collection
Returns a collection containing only the last item in the input collection. Will return an empty collection if the input collection has no items.

5.3.5. tail() : collection
Returns a collection containing all but the first item in the input collection. Will return an empty collection if the input collection has no items, or only one item.

5.3.6. skip(num : integer) : collection
Returns a collection containing all but the first num items in the input collection. Will return an empty collection if there are no items remaining after the indicated number of items have been skipped, or if the input collection is empty. If num is less than or equal to zero, the input collection is simply returned.

5.3.7. take(num : integer) : collection
Returns a collection containing the first num items in the input collection, or less if there are less than num items. If num is less than or equal to 0, or if the input collection is empty ({ }), take returns an empty collection.

5.4. Combining
5.4.1. | (union collections)
Merge the two collections into a single collection, eliminating any duplicate values (using equals (=)) to determine equality). Unioning an empty collection to a non-empty collection will return the non-empty collection with duplicates eliminated. There is no expectation of order in the resulting collection.

5.4.2. combine(other : collection) : collection
Merge the input and other collections into a single collection without eliminating duplicate values. Combining an empty collection with a non-empty collection will return the non-empty collection. There is no expectation of order in the resulting collection.

5.5. Conversion
The functions in this section operate on collections with a single item. If there is more than one item, or an incompatible item, the evaluation of the expression will end and signal an error to the calling environment.

To use these functions over a collection with multiple items, one may use filters like where() and select():

Patient.name.given.select(substring(1))
This example returns a collection containing the first character of all the given names for a patient.

5.5.1. iif(criterium: expression, true-result: collection [, otherwise-result: collection]) : collection
If criterium is true, the function returns the value of true-result parameter.

If criterium is false or an empty collection, the function returns otherwise-result, unless the optional otherwise-expression is not given, in which case the function returns an empty collection.

5.5.2. toInteger() : integer
If the input collection contains a single item, this function will return a single integer if:

the item in the input collection is an integer

the item in the input collection is a string and is convertible to an integer

the item is a boolean, where true results in a 1 and false results in a 0.

If the item is not one the above types, the evaluation of the expression will end and signal an error to the calling environment.

If the item is a string, but the string is not convertible to an integer (using the regex format (\\+|-)?\d+), the evaluation of the expression will end and signal an error to the calling environment.

In all other cases, the function will return an empty collection.

5.5.3. toDecimal() : decimal
If the input collection contains a single item, this function will return a single decimal if:

the item in the input collection is an integer or decimal

the item in the input collection is a string and is convertible to a decimal

the item is a boolean, where true results in a 1.0 and false results in a 0.0.

If the item is not one of the above types, the evaluation of the expression will end and signal an error to the calling environment.

If the item is a string, but the string is not convertible to a decimal (using the regex format (\\+|-)?\d+('.' \d+)?), the evaluation of the expression will end and signal an error to the calling environment.

In all other cases, the function will return an empty collection.

5.5.4. toString() : string
If the input collection contains a single item, this function will return a single string if:

the item in the input collection is a string

the item in the input collection is an integer, decimal, time or dateTime the output will contain its string representation

the item is a boolean, where true results in 'true' and false in 'false'.

If the item is not one of the above types, the evaluation of the expression will end and signal an error to the calling environment.

The string representation uses the following formats:

Type	Representation
boolean

true or false

integer

(\\+|-)?\d+

decimal

(\\+|-)?\d+(.\d+)?

quantity

(\\+|-)?\d+(.\d+)? '<unit>'

dateTime

YYYY-MM-DDThh:mm:ss.fff(+/-)hh:mm

time

Thh:mm:ss.fff(+/-)hh:mm

Note that for partial dates and times, the result will only be specified to the level of precision in the value being converted.

In all other cases, the function will return an empty collection.

5.6. String Manipulation
The functions in this section operate on collections with a single item. If there is more than one item, or an item that is not a string, the evaluation of the expression will end and signal an error to the calling environment.

5.6.1. indexOf(substring : string) : integer
If the input collection contains a single item of type string, will return the 0-based index of the first position this substring is found in the input string, or -1 if it is not found. If the substring is an empty string, the function returns 0.

5.6.2. substring(start : integer [, length : integer]) : string
If the input collection contains a single item of type string, it returns a collection with the part of the string starting at position start (zero-based). If length is given, will return at most length number of characters from the input string.

If start lies outside the length of the string, the function returns an empty collection. If there are less remaining characters in the string than indicated by length, the function returns just the remaining characters.

5.6.3. startsWith(prefix : string) : boolean
If the input collection contains a single item of type string, the function will return true when the input string starts with the given prefix. Also returns true when prefix is the empty string.

5.6.4. endsWith(suffix : string) : boolean
If the input collection contains a single item of type string, the function will return true when the input string ends with the given suffix. Also returns true when suffix is the empty string.

5.6.5. contains(substring : string) : boolean
If the input collection contains a single item of type string, the function will return true when the given substring is a substring of the input string. Also returns true when substring is the empty string.

5.6.6. replace(pattern : string, substitution : string) : string
If the input collection contains a single item of type string, the function will return the input string with all instances of pattern replaced with substitution. If the substitution is the empty string, the instances of the pattern are removed from the input string. If the pattern is the empty string, every character in the input string is surrounded by the substitution, e.g. 'abc'.replace('','x') becomes 'xaxbxcx'.

5.6.7. matches(regex : string) : boolean
If the input collection contains a single item of type string, the function will return true when the value matches the given regular expression. Regular expressions should function consistently, regardless of any culture- and locale-specific settings in the environment, should be case-sensitive, use 'single line' mode and allow Unicode characters.

5.6.8. replaceMatches(regex : string, substitution: string) : string
If the input collection contains a single item of type string, the function will match the input using the regular expression in regex and replace each match with the substitution string. The substitution may refer to identified match groups in the regular expression.

This example of replace() will convert a string with a date formatted as MM/dd/yy to dd-MM-yy:

'11/30/1972'.replace('\\b(?<month>\\d{1,2})/(?<day>\\d{1,2})/(?<year>\\d{2,4})\\b',
       '${day}-${month}-${year}')
Note: Platforms will typically use native regular expression implementations. These are typically fairly similar, but there will always be small differences. As such, FHIRPath does not prescribe a particular dialect, but recommends the use of the dialect defined by as part of XML Schema 1.1 as the dialect most likely to be broadly supported and understood.

5.6.9. length() : integer
If the input collection contains a single item of type string, the function will return the length of the string. If the input collection is empty ({ }), the result is empty.

5.7. Tree navigation
5.7.1. children() : collection
Returns a collection with all immediate child nodes of all items in the input collection. Note that the ordering of the children is undefined and using operations like first() on the result may return different results on different platforms.

5.7.2. descendants() : collection
Returns a collection with all descendant nodes of all items in the input collection. The result does not include the nodes in the input collection themselves. Is a shorthand for repeat(children()). Note that the ordering of the children is undefined and using operations like first() on the result may return different results on different platforms.

Note: Many of these functions will result in a set of nodes of different underlying types. It may be necessary to use ofType() as described in the previous section to maintain type safety. See section 8 for more information about type safe use of FHIRPath expressions.

5.8. Utility functions
5.8.1. trace(name : string) : collection
Add a string representation of the input collection to the diagnostic log, using the parameter name as the name in the log. This log should be made available to the user in some appropriate fashion. Does not change the input, so returns the input collection as output.

5.8.2. today() : dateTime
Returns a dateTime containing the current date.

5.8.3. now() : dateTime
Returns a dateTime containing the current date and time, including timezone.

6. Operations
Operators are allowed to be used between any kind of path expressions (e.g. expr op expr). Like functions, operators will generally propagate an empty collection in any of their operands. This is true even when comparing two empty collections using the equality operators, e.g.

{} = {}
true > {}
{} != 'dummy'
all result in {}.

6.1. Equality
6.1.1. = (Equals)
Returns true if the left collection is equal to the right collection:

If both operands are collections with a single item:

For primitives:

string: comparison is based on Unicode values

integer: values must be exactly equal

decimal: values must be equal, trailing zeroes are ignored

boolean: values must be the same

dateTime: must be exactly the same, respecting the timezone (though +24:00 = +00:00 = Z)

time: must be exactly the same, respecting the timezone (though +24:00 = +00:00 = Z)

If a time or dateTime has no indication of timezone, the timezone of the evaluating machine is assumed.

For complex types, equality requires all child properties to be equal, recursively.

If both operands are collections with multiple items:

Each item must be equal

Comparison is order dependent

Otherwise, equals returns false.

Note that this implies that if the collections have a different number of items to compare, the result will be false.

Typically, this operator is used with single fixed values as operands. This means that Patient.telecom.system = 'phone' will return false if there is more than one telecom with a use. Typically, you’d want Patient.telecom.where(system = 'phone')

If one or both of the operands is the empty collection, this operation returns an empty collection.

For dateTime and time comparisons with partial values (e.g. dateTimes specified only to the day, or times specified only to the hour), the comparison returns empty ({ }), not false.

6.1.2. ~ (Equivalent)
Returns true if the collections are the same. In particular, comparing empty collections for equivalence { } ~ { } will result in true.

If both operands are collections with a single item:

For primitives

string: the strings must be the same while ignoring case and normalizing whitespace.

integer: exactly equal

decimal: values must be equal, comparison is done on values rounded to the precision of the least precise operand. Trailing zeroes are ignored in determining precision.

dateTime and time: values must be equal, except that for partial date/time values, the comparison returns false, not empty ({ }). If one operand has less precision than the other, comparison is done at the lowest precision.

boolean: the values must be the same

For complex types, equivalence requires all child properties to be equivalent, recursively.

If both operands are collections with multiple items:

Each item must be equivalent

Comparison is not order dependent

Note that this implies that if the collections have a different number of items to compare, the result will be false.

6.1.3. != (Not Equals)
The inverse of the equals operator.

6.1.4. !~ (Not Equivalent)
The inverse of the equivalent operator.

6.2. Comparison
The comparison operators are defined for strings, integers, decimals, dateTimes and times.

If one or both of the arguments is an empty collection, a comparison operator will return an empty collection.

Both arguments must be collections with single values, and the evaluator will throw an error if either collection has more than one item.

Both arguments must be of the same type, and the evaluator will throw an error if the types differ.

When comparing integers and decimals, the integer will be converted to a decimal to make comparison possible.

String ordering is strictly lexical and is based on the Unicode value of the individual characters.

For partial date/time values, the comparison is performed to the highest precision specified in both values.

6.2.1. > (Greater Than)
6.2.2. < (Less Than)
6.2.3. <= (Less or Equal)
6.2.4. >= (Greater or Equal)
6.3. Types
6.3.1. is
If the left operand is a collection with a single item and the second operand is a type identifier, this operator returns true if the type of the left operand is the type specified in the second operand, or a subclass thereof. If the identifier cannot be resolved to a valid type identifier, the evaluator will throw an error. If the input collections contains more than one item, the evaluator will throw an error. In all other cases this function returns the empty collection.

Patient.contained.all($this is Patient implies age > 10)
This example returns true if for all the contained resources, if the contained resource is of type Patient, then the age is greater than ten.

6.3.2. as
If the left operand is a collection with a single item and the second operand is an identifier, this function returns the value of the left operand if it is of the type specified in the second operand, or a subclass thereof. If the identifier cannot be resolved to a valid type identifier, the evaluator will throw an error. If there is more than one item in the input collection, the evaluator will throw an error. Otherwise, this operator returns the empty collection.

6.4. Collections
6.4.1. | (union collections)
Merge the two collections into a single collection, eliminating any duplicate values (using equals (=)) to determine equality). Unioning an empty collection to a non-empty collection will return the non-empty collection with duplicates eliminated. There is no expectation of order in the resulting collection.

6.4.2. in (membership)
If the left operand is a collection with a single item, this operator returns true if the item is in the right operand using equality semantics. If the left-hand side of the operator is the empty collection is empty, the result is empty, if the right-hand side is empty, the result is false. If the left operand has multiple items, an exception is thrown.

6.4.3. contains (containership)
If the right operand is a collection with a single item, this operator returns true if the item is in the left operand using equality semantics. This is the inverse operation of in.

6.5. Boolean logic
For all boolean operators, the collections passed as operands are first evaluated as booleans (as described in Boolean Evaluation of Collections). The operators then use three-valued logic to propagate empty operands.

Note: To ensure that FHIRPath expressions can be freely rewritten by underlying implementations, there is no expectation that an implementation respect short-circuit evaluation. With regard to performance, implementations may use short-circuit evaluation to reduce computation, but authors should not rely on such behavior, and implementations must not change semantics with short-circuit evaluation. If a condition is needed to ensure correct evaluation of a subsequent expression, the iif() function should be used to guarantee that the condition determines whether evaluation of an expression will occur at run-time.

6.5.1. and
Returns true if both operands evaluate to true, false if either operand evaluates to false, and empty collection ({ }) otherwise:

 	true	false	empty ({ })
true

true

false

empty ({ })

false

false

false

false

empty ({ })

empty ({ })

false

empty ({ })

6.5.2. or
Returns false if both operands evaluate to false, true if either operand evaluates to true, and empty ({ }) otherwise:

 	true	false	empty ({ })
true

true

true

true

false

true

false

empty ({ })

empty ({ })

true

empty ({ })

empty ({ })

6.5.3. xor
Returns true if exactly one of the operands evaluates to true, false if either both operands evaluate to true or both operands evaluate to false, and the empty collection ({ }) otherwise:

 	true	false	empty ({ })
true

false

true

empty ({ })

false

true

false

empty ({ })

empty ({ })

empty ({ })

empty ({ })

empty ({ })

6.5.4. implies
If the left operand evaluates to true, this operator returns the boolean evaluation of the right operand. If the left operand evaluates to false, this operator returns true. Otherwise, this operator returns true if the right operand evaluates to true, and the empty collection ({ }) otherwise.

 	true	false	empty ({ })
true

true

false

empty ({ })

false

true

true

true

empty ({ })

true

empty ({ })

empty ({ })

The implies operator is useful for testing conditionals. For example, if a given name is present, then a family name must be as well:

Patient.name.given.exists() implies Patient.name.family.exists()
6.6. Math
The math operators require each operand to be a single element. Both operands must be of the same type, each operator below specifies which types are supported.

If there is more than one item, or an incompatible item, the evaluation of the expression will end and signal an error to the calling environment.

As with the other operators, the math operators will return an empty collection if one or both of the operands are empty.

6.6.1. * (multiplication)
Multiplies both arguments (numbers only)

6.6.2. / (division)
Divides the left operand by the right operand (numbers only).

6.6.3. + (addition)
For integer and decimal, add the operands. For strings, concatenates the right operand to the left operand.

6.6.4. - (subtraction)
Subtracts the right operand from the left operand (numbers only).

6.6.5. div
Performs truncated division of the left operand by the right operand (numbers only).

6.6.6. mod
Computes the remainder of the truncated division of its arguments (numbers only).

6.6.7. & (string concatenation)
For strings, will concatenate the strings, where an empty operand is taken to be the empty string. This differs from + on two strings, which will result in an empty collection when one of the operands is empty.

6.7. Date/Time Arithmetic
Date and time arithmetic operators are used to add time-valued quantities to date/time values. The left operand must be a dateTime or time value, and the right operand must be a quantity with a time-valued unit:

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
Returns the value of the given dateTime or time, incremented by the time-valued quantity, respecting variable length periods for calendar years and months.

For dateTime values, the quantity unit must be one of: years, months, days, hours, minutes, seconds, or milliseconds (or an equivalent unit), or an error is raised.

For time values, the quantity unit must be one of: hours, minutes, seconds, or milliseconds (or an equivalent unit), or an error is raised.

For partial date/time values, the operation is performed by converting the time-valued quantity to the highest precision in the partial (removing any decimal value off) and then adding to the date/time value. For example:

@2014 + 24 months
This expression will evaluate to the value @2016 even though the date/time value is not specified to the level of precision of the time-valued quantity.

6.7.2. - (subtraction)
Returns the value of the given dateTime or time, decremented by the time-valued quantity, respecting variable length periods for calendar years and months.

For dateTime values, the quantity unit must be one of: years, months, days, hours, minutes, seconds, milliseconds (or an equivalent unit), or an error is raised.

For time values, the quantity unit must be one of: hours, minutes, seconds, or milliseconds (or an equivalent unit), or an error is raised.

For partial date/time values, the operation is performed by converting the time-valued quantity to the highest precision in the partial (removing any decimal value off) and then subtracting from the date/time value. For example:

@2014 - 24 months
This expression will evaluate to the value @2012 even though the date/time value is not specified to the level of precision of the time-valued quantity.

6.8. Operator precedence
Precedence of operations, in order from high to low:

#01 . (path/function invocation)
#02 [] (indexer)
#03 unary + and -
#04: *, /, div, mod
#05: +, -, &
#06: |
#07: >, <, >=, <=
#08: is, as
#09: =, ~, !=, !~
#10: in, contains
#11: and
#12: xor, or
#13: implies
As customary, expressions may be grouped by parenthesis (()).

7. Lexical Elements
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

7.1. Whitespace
FHIRPath defines tab, space, and return as whitespace, meaning they are only used to separate other tokens within the language. Any number of whitespace characters can appear, and the language does not use whitespace for anything other than delimiting tokens.

7.2. Comments
FHIRPath defines two styles of comments, single-line, and multi-line. A single-line comment consists of two forward slashes, followed by any text up to the end of the line:

2 + 2 // This is a single-line comment
To begin a multi-line comment, the typical forward slash-asterisk token is used. The comment is closed with an asterisk-forward slash, and everything enclosed is ignored:

/*
This is a multi-line comment
Any text enclosed within is ignored
*/
7.3. Literals
Literals provide for the representation of values within FHIRPath. The following types of literals are supported:

Literal	Description
Empty ({ })

The empty collection

boolean

The boolean literals (true and false)

integer

Sequences of digits in the range 0-2^32-1

decimal

Sequences of digits with a decimal point, in the range 0.0..1028-10-8

string

Strings of any character enclosed within single-ticks (')

dateTime

The at-symbol (@) followed by a date time

time

The at-symbol (@) followed by a time

quantity

An integer or decimal literal followed by a datetime precision specifier, or a UCUM unit specifier

For a more detailed discussion of the use and semantics of literals within expressions, refer to the Expressions section above.

7.4. Symbols
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

= != ⇐ < > >=

Comparison operators for comparing values

`+ - * /

&`

7.5. Keywords
Keywords are tokens that are recognized by the parser and used to build the various language constructs. FHIRPath defines the following keywords:

$this

hour

month

and

implies

or

as

in

second

contains

is

true

day

millisecond

week

div

minute

xor

false

mod

year

In general, keywords within FHIRPath are also considered reserved words, meaning that it is illegal to use them as identifiers. If necessary, identifiers that clash with a reserved word can be quoted.

7.6. Identifiers
Identifiers are used as labels to allow expressions to reference elements such as model types and properties. FHIRPath supports two types of identifiers, simple and quoted.

A simple identifier is any alphabetical character or an underscore, followed by any number of alpha-numeric characters or underscores. For example, the following are all valid simple identifiers:

Patient
_id
valueDateTime
A quoted identifier is any sequence of characters enclosed in double-quotes ("):

"QI-Core Patient"
"US-Core Diagnostic Request"
"us-zip"
The use of double-quotes allows identifiers to contains spaces, commas, and other characters that would not be allowed within simple identifiers. This allows identifiers to be more descriptive, and also enables expressions to reference models that have property or type names that are not valid simple identifiers.

FHIRPath escape sequences for strings also work for quoted identifiers:

Escape	Character
\'

Single-quote

\"

Double-quote

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

7.7. Case-Sensitivity
FHIRPath is a case-sensitive language, meaning that case is considered when matching keywords in the language. However, because FHIRPath can be used with different models, the case-sensitivity of type and property names is defined by each model.

8. Environment variables
A token introduced by a % refers to a value that is passed into the evaluation engine by the calling environment. Using environment variables, authors can avoid repetition of fixed values and can pass in external values and data.

The following environmental values are set for all contexts:

%ucum       // (string) url for ucum
%context	// The original node that was passed to the evaluation engine before starting evaluation
Implementers should note that using additional environment variables is a formal extension point for the language. Various usages of FHIRPath may define their own externals, and implementers should provide some appropriate configuration framework to allow these constants to be provided to the evaluation engine at run time. E.g.:

%"us-zip" = '[0-9]{5}(-[0-9]{4}){0,1}'
Note that the identifier portion of the token is allowed to be either a simple identifier (as in %ucum), or a quoted identifier to allow for alternative characters (as in %"us-zip").

Note also that these tokens are not restricted to simple types, and they may have values that are not defined fixed values known prior to evaluation at run-time, though there is no way to define these kind of values in implementation guides.

9. Reflection
FHIRPath supports reflection to provide the ability for expressions to access type information describing the structure of values. The type() function returns the type information for each element of the input collection, using one of the following concrete subtypes of TypeInfo:

For primitive types, the result is a SimpleTypeInfo:

SimpleTypeInfo { name: string, baseType: SimpleTypeInfo }
For class types, the result is a ClassInfo:

ClassInfoElement { name: string, type: TypeInfo }
ClassInfo { name: string, baseType: TypeInfo, element: List<ClassInfoElement> }
For collection types, the result is a ListTypeInfo:

ListTypeInfo { elementType: TypeInfo }
And for anonymous types, the result is a TupleTypeInfo:

TupleTypeInfoElement { name: string, type: TypeInfo }
TupleTypeInfo { element: List<TupleTypeInfoElement> }
Note: These structures are a subset of the abstract metamodel used by the Clinical Quality Language Tooling.

10. Type safety and strict evaluation
Strongly typed languages are intended to help authors avoid mistakes by ensuring that the expressions describe meaningful operations. For example, a strongly typed language would typically disallow the expression:

1 + 'John'
because it performs an invalid operation, namely adding numbers and strings. However, there are cases where the author knows that a particular invocation may be safe, but the compiler is not aware of, or cannot infer, the reason. In these cases, type-safety errors can become an unwelcome burden, especially for experienced developers.

As a result, FHIRPath defines a strict option that allows an execution environment to determine how much type safety should be applied. With strict enabled, FHIRPath behaves as a traditional strongly-typed language, whereas without strict, it behaves as a traditional dynamically-typed language.

For example, since some functions and most operators will only accept a single item as input, and throw an exception otherwise:

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

The strict option provides a mode in which the author of the FHIRPath statement is protected against such cases by employing strict typing. Based on the definition of the operators and functions and given the type of input, a compiler can trace the statement and determine whether "unsafe" situations can occur.

Unsafe uses are:

A function that requires an input collection with a single item is called on an output that is not guaranteed to have only one item.

A function is passed an argument that is not guaranteed to be a single value.

A function is passed an input value or argument that is not of the expected type

An operator that requires operands to be collections with a single item is called with arguments that are not guaranteed to have only one item.

An operator has operands that are not of the expected type

Equality operators are used on operands that are not both collections or collections containing a single items.

There are a few constructs in the FHIRPath language where the compiler cannot trace the type, and should issue a warning to the user when doing "strict" evaluation:

The children() and descendants() functions

The resolve() function

A member which is polymorphic (e.g. a choice[x] type in FHIR)

Authors can use the as operator directly after such constructs to inform the compiler of the expected type, so that strict type-checking can continue.

In strict mode, when the compiler finds places where a collection of multiple items can be present while just a single item is expected, the author will need to make explicit how repetitions are dealt with. Depending on the situation one may:

Use first(), last() or indexer ([ ]) to select a single item

Use select() and where() to turn the expression into one that evaluates each of the repeating items individually (as in the examples above)

Use single() to return either the single item or else an empty collection. This is especially useful when using FHIRPath to formulate invariants: in cases where single items are considered the "positive" or "true" situation, single() will return an empty collection, so the invariant will evaluate to the empty collection (or false) in any other circumstance.

[27-1-2016 20:53:40] Grahame Grieve: I still disagree with the way that the strict evaluation
section is framed. The issue should be described, and then possible approaches defined,
including type level evaluation for warnings. I'm not sure when a strict mode on evaluation would actually be appropriate or
whether strict must be applied purely based on the type definitions, or whether an evaluation
engine is allowed to be aware of contextual constraints in addition to the type definitions
The as operator will not be useable if the parent, children or descendants contain backbone elements (which they will). This might not be a big problem, e.g.

Questionnaire.descendants().question
will really always just select the "anonymous" child called "question" and thus the Question complex element, but we will not get any typesafety again until we do

Questionnaire.descendants().question.concept.as('Coding').etc
Appendix A: Formal Specifications
A.1. Formal Syntax
The formal syntax for FHIRPath is specified as an Antlr 4.0 grammar file (g4) and included in this specification at the following link:

grammar.html

A.2. Model Information
The model information returned by the reflection function type() is specified as an XML Schema document (xsd) and included in this specification at the following link:

modelinfo.xsd

Note
The model information file included here is not a normative aspect of the FHIRPath specification. It is the same model information file used by the Clinical Quality Framework Tooling and is included for reference as a simple formalism that meets the requirements described in the normative Reflection section above.
As discussed in the section on case-sensitivity, each model used within FHIRPath determines whether or not identifiers in the model are case-sensitive. This information is provided as part of the model information and tooling should respect the case-sensitive settings for each model.

A.3. URI and Media Types
To uniquely identify the FHIRPath language, the following URI is defined:

http://hl7.org/fhirpath

In addition, a media type is defined to support describing FHIRPath content:

text/fhirpath

Appendix B: Use of FHIRPath in HL7 FHIR
FHIRPath is used in five places in the FHIR specifications: - search parameter paths - used to define what contents the parameter refers to - slicing discriminator - used to indicate what element(s) define uniqueness - invariants in ElementDefinition, used to apply co-occurrence and other rules to the contents - error message locations in OperationOutcome - URL templates in Smart on FHIR’s cds-hooks - may be used for Patch in the future

As stated in the introduction, FHIRPath uses a tree model that abstracts away the actual underlying datamodel of the data being queried. For FHIR, this means that the contents of the resources and data types as described in the Logical views (or the UML diagrams) are used as the model, rather than the JSON and XML formats, so specific xml or json features are not visible to the FHIRPath language (such as comments and the split representation of primitives).

More specifically:

A FHIRPath may optionally start with a full resource name

Elements of datatypes and resources are used as the name of the nodes which can be navigated over, except for choice elements (ending with '[x]'), see below.

The contained element node does not have the name of the Resource as its first and only child (instead it directly contains the contained resource’s children)

There is no difference between an attribute and an element

Repeating elements turn into multiple nodes with the same name

B.1. Polymorphism in FHIR
FHIR has the notion of choice elements, where elements can be one of multiple types, e.g. Patient.deceased[x]. In actual instances these will be present as either Patient.deceasedBoolean or Patient.deceasedDateTime. In FHIRPath choice elements are labeled according to the name without the '[x]' suffix, and children can be explicitly filtered using the as operation:

(Observation.value as Quantity).unit
B.2. Using FHIR types in expressions
The evaluation engine will automatically convert the value of FHIR types representing primitives to FHIRPath types when they are used in expression in the following fashion:

FHIR primitive type	FHIRPath type
boolean

boolean

string, uri, code, oid, id, uuid, sid, markdown, base64Binary

string

integer, unsignedInt, positiveInt

integer

decimal

decimal

date, dateTime, instant

dateTime

time

time

Note that FHIR primitives may contain extensions, so that the following expressions are not mutually exclusive:

Patient.name.given = 'Ewout'         // value of Patient.name.given as a string
Patient.name.given.extension.first().value = true   // extension of the primitive value
B.3. Additional functions
FHIR adds (backwards compatible) functionality to the common set of functions:

B.3.1. extension(url : string) : collection
Will filter the focus for items named "extension" with the given url. This is a syntactical shortcut for .extension.where(url = string), but is simpler to write. Will return an empty collection if the focus is empty or the url is empty.

B.3.2. trace(name : string) : collection
When FHIRPath statements are used in an invariant, the log contents should be added to the error message constructed when the invariant is violated. For example:

"SHALL have a local reference if the resource is provided inline (url: height; ids: length,weight)"

from

"reference.startsWith('#').not()
    or ($context.reference.substring(1).output('url') in $resource.contained.id.output('ids'))"
B.3.3. resolve() : collection
For each item in the collection, if it is a string, locate the target of the reference, and add it to the resulting collection. If the item is not a string, the item is ignored and nothing is added to the output collection.

The items in the collection may also represent a Reference, in which case the Reference.reference is resolved.

If fetching the resource fails, the failure message is added to the output collection.

B.3.4. ofType(type : identifier) : collection
In FHIR, only concrete core types are allowed as an argument. All primitives are considered to be independent types (so markdown is not a subclass of string). Profiled types are not allowed, so to select SimpleQuantity one would pass Quantity as an argument.

B.3.5. elementDefinition() : collection
Returns the FHIR element definition information for each element in the input collection.

B.3.6. slice(structure : string, name : string) : collection
Returns the given slice as defined in the given structure definition. The structure argument is a uri that resolves to the structure definition, and the name must be the name of a slice within that structure definition. If the structure cannot be resolved, or the name of the slice within the resolved structure is not present, an error is thrown.

For every element in the input collection, if the resolved slice is present on the element, it will be returned. If the slice does not match any element in the input collection, or if the input collection is empty, the result is an empty collection ({ }).

B.3.7. checkModifiers([{modifier : string}]) : collection
For each element in the input collection, verifies that there are no modifying extensions defined other than the ones given by the modifier argument. If the check passes, the input collection is returned. Otherwise, an error is thrown.

B.3.8. conformsTo(structure : string) : boolean
Returns true if the single input element conforms to the profile specified by the structure argument, and false otherwise. If the structure cannot be resolved to a valid profile, an error is thrown. If the input contains more than one element, an error is thrown. If the input is empty, the result is empty.

B.4. Changes to operators
B.4.1. ~ (Equivalence)
Equivalence works in exactly the same manner, but with the addition that for complex types, equality requires all child properties to be equal, except for "id" elements.

B.4.2. in (Membership)
Membership works in the same way, with the added capability that if the right argument is a string, it is resolved as a uri to a value set and the operation performs value set membership.

Note that implementations are encouraged to make use of a terminology service to provide this functionality.

For example:

Observation.component.where(code in 'http://hl7.org/fhir/ValueSet/observation-vitalsignresult')
This expression returns components that have a code that is a member of the observation-vitalsignresult valueset.

B.5. Environment variables
The FHIR specification adds support for additional environment variables:

The following environmental values are set for all contexts:

%sct        // (string) url for snomed ct
%loinc      // (string) url for loinc
%"vs-[name]" // (string) full url for the provided HL7 value set with id [name]
%"ext-[name]" // (string) full url for the provided HL7 extension with id [name]
%resource	// The original resource current context is part of. When evaluating a datatype, this would be the resource the element is part of. Do not go past a root resource into a bundle, if it is contained in a bundle.

// Note that the names of the `vs-` and `ext-` constants are quoted (just like paths) to allow "-" in the name.
For example:

Observation.component.where(code in %"vs-observation-vitalsignresult")
This expression returns components that have a code that is a member of the observation-vitalsignresult valueset.

Note: Implementation Guides are allowed to define their own externals, and implementers should provide some appropriate configuration framework to allow these constants to be provided to the evaluation engine at run time. E.g.:

%"us-zip" = '[0-9]{5}(-[0-9]{4}){0,1}'
Authors of Implementation Guides should be aware that adding specific environment variables restricts the use of the FHIRPath to their particular context.

Note that these tokens are not restricted to simple types, and they may have fixed values that are not known before evaluation at run-time, though there is no way to define these kind of values in implementation guides.

Appendix C: Use of FHIRPath in Clinical Quality Language (CQL)
Clinical Quality Language is being extended to use FHIRPath as its core path language, in much the same way that XQuery uses XPath to represent paths within queries. In particular, the following extensions to CQL are proposed:

C.1. Path Traversal
When a path expression involves an element with multiple cardinality, the expression is considered short-hand for an equivalent query invocation. For example:

Patient.telecom.use
is allowed, and is considered a short-hand for the following query expression:

Patient.telecom X where X.use is not null return X.use
Given a patient with multiple telecom entries, the above query will return a list containing the use element from each entry. Note that the restriction is required as it ensures that the resulting list will not contain any null elements. In addition, if the element itself is list-valued, the result is expanded:

Patient.name.given
is short-hand for:

expand (Patient.name X where X.given is not null return (X.given Y where Y is not null))
In this case, given a patient with multiple names, each of which has multiple givens, this will return a single list containing all the given names in all the names.

C.2. Constants and Contexts
FHIRPath has the ability to reference contexts (using the $ prefix) and environment-defined variables (using the % prefix). Within CQL, these contexts and environment-defined variables are added to the appropriate scope (global for environment-variables, local for contexts) without the prefix. Additionally, because the % prefix is optional, it is not required to access environment-defined variables within CQL.

C.3. Additional Operators
The following additional operators are being added to CQL:

~, !~ - Equivalent operators (formerly matches in CQL)

!= - As a replacement for <>

implies - Logical implication

| - As a synonym for union

C.4. Method-style Invocation
One of the primary syntactic features of FHIRPath is the ability to "invoke" a function on a collection. For example:

Patient.name.given.substring(3)
The CQL syntax is being extended to support this style of invocation, but as a short-hand for an equivalent CQL statement for each operator. For example:

stringValue.substring(3, 5)
is allowed, and is considered a short-hand for the following CQL expression:

Substring(stringValue, 3, 5)
For most functions, this short-hand is a simple rewrite, but for contextual functions such as where() and select(), this rewrite must preserve the context semantics:

Patient.name.where(given = 'John')
is short-hand for:

Patient.name N where N.given = 'John'
C.5. Strict Evaluation
Because CQL is a type-safe language, embedded FHIRPath expressions should be compiled in strict mode. However, to enable the use of FHIRPath in loose mode, an implicit conversion from a list of elements to an element is added. This implicit conversion is implemented as an invocation of singleton from, ensuring that if the list has multiple elements at run-time an error will be thrown.

In addition, the underlying Expression Logical Model (ELM) is being extended to allow for dynamic invocation. A Dynamic type is introduced with appropriate operators to support run-time invocation where necessary. However, these operators are introduced as an additional layer on top of core ELM, and CQL compiled with the strict option will never produce expressions containing these elements. This avoids placing additional implementation burden on systems that do not need dynamic capabilities.

Appendix D: Use of FHIRPath on HL7 Version 2 messages
FHIRPath can be used against HL7 v2 messages. This UML diagram summarises the object model on which the FHIRPath statements are written:

Object Model for HL7 v2

In this Object Model:

The object graph always starts with a message.

Each message has a list of segments.

In addition, Abstract Message Syntax is available through the groups() operation, for use where the message follows the Abstract Message Syntax sufficiently for the parser to reconcile the segment list with the structure.

The names of the groups are the names published in the specification, e.g. 'PATIENT_OBSERVATION' (with spaces, where present, replaced by underscores. In case of doubt, consult the v2 XML schemas).

Each Segment has a list of fields, which each have a list of "Cells". This is necessary to allow for repeats, but users are accustomed to just jumping to Element - use the operation elements() which returns all repeats with the given index.

A "cell" can be either an Element, a Component or a Sub-Components. Elements can contain Components, which can contain Sub-Components. Sub-Sub-Components are not allowed.

Calls may have a simple text content, or a series of (sub-)components. The simple() operation returns either the text, if it exists, or the return value of simple() from the first component

A v2 data type (e.g. ST, SN, CE etc) is a profile on Cell that specifies whether it has simple content, or complex content.

todo: this object model doesn’t make provision for non-syntax escapes in the simple content (e.g. \.b\).

all the lists are 1 based. That means the first item in the list is numbered 1, not 0.

Some example queries:

Message.segment.where(code = 'PID').field[3].element.first.simple()
Get the value of the first component in the first repeat of PID-3

Message.segment[2].elements(3).simple()
Get a collection with is the string values of all the repeats in the the 3rd element of the 2nd segement. Typically, this assumes that there is no repeats, and so this is a simple value

Message.segment.where(code = 'PID').field[3].element.where(component[4].value = 'MR').simple()
Pick out the MR number from PID-3 (assuming, in this case, that there’s only one PID segment in the message. No good for an A17). Note that this returns the whole Cell - e.g. |value^^MR|, though often more components will be present)

Message.segment.where(code = 'PID').elements(3).where(component[4].value = 'MR').component[1].text
Same as the last, but pick out just the MR value

Message.group("PATIENT").group("PATIENT_OBSERVATION").item.as(Segment)
  .where(code = 'OBX' and elements(2).any(components(2) = 'LN')))
return any OBXs from the patient observations (and ignore others e.g. in a R01 message) segments that have LOINC codes. Note that if the parser cannot properly parse the Abstract Message Syntax, group() must fail with an error message.

Appendix E: References
[ANTLR] Another Tool for Language Recognition (ANTLR) http://www.antlr.org/

[CQL] HL7 Cross-Paradigm Specification: Clinical Quality Language, Release 1, DSTU Release 1.1. http://www.hl7.org/implement/standards/product_brief.cfm?product_id=400

[XMLRE] Regular Expressions. https://www.w3.org/TR/xmlschema11-2/#regexs

[PCRE] Pearl-Compatible Regular Expressions. http://www.pcre.org/
