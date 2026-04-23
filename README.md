# eXtensible Classification Notation (XCN)

# Abstract

XCN is a binary exchange notation system that relies heavily on [UTF-8](#utf-8-rfc-3629) text-based data.  It is largely a derivative of [EJCN (Extensable JSON Classification Notation)](#ejcn-extensable-json-classification-notation), [XML](#xml-wc3), [JSON](#json-rfc-8259), and their predecessors, including [HTML](#html) and the [Apache HTTP Server configuration files](#apache-http-server-configuration-files).  Unlike [JSON](#ejcn-extensable-json-classification-notation) and [XML](#xml-wc3), which are [context-free grammars](#context-free-grammers) XCN is a hybrid context-sensitive and [context-free grammar](#context-free-grammers).  In addition, XCN's goals were to improve upon JSON (the modern de-facto data interchange standard) by adding the following improvements;

- pure binary data
- comments
- arbitrary data encodings (aka. embedded JSON, XML, HTML, Google ProtocolBuffers, etc)
- a fix to JSON's number serialization issues

Finally, XCN also attempts to improve upon [Parquet](#parquet) and other [columnar data storage techniques](#row-vs-column-formats) through data segment/row scanning optimizations. 

# Introduction

In order to keep the mixing pot of features straight in XCN in as simple a manner as possible, there are two major structures involved. First, the super-structure which was originally envisioned by EJCN consists of a recursive header, body format.  The super-structure is context-sensitive. The super-structure is recursive because the body may contain one or more super-structures.  When one or more super structures exist inside of a super-structure body, this is known as a super-structure-sequence.  Second the XCN grammar-structure is used.  The XCN grammar structure is [context-free](#context-free-grammers).

## The Super-Structure

The XCN super-structure MUST be comprised of a header-section and then a body-section.  XCN's super-structure body-sections MAY include anything from pure binary to HTML.  However, XCN super-structure header-section MUST use XCN's grammar structure.  In addition, the header-section MUST be comprised of XCNLineSegments, comments, or both.

### XCN Header Section Details



## The Grammar-Structure

Similar to JSON, XCN grammar is a sequence of tokens.  XCN tokens are separated into simple and classified token segments. However, unlike JSON, the tokens are segments of characters separated by whitespace.  In addition, token segments MAY consist of multiple characters.  Finally, some tokens are classified under the following base classification [XCNObjects](#xcn-objects), [XCNLineSegments](#xcn-line-segments), [XCNTrees](#xcn-trees), and [XCNTables](#xcn-tables) as follows;

### XCN Package Names

XCN package names are generally derived from the reverse order of a DNS domain name.  However, non-DNS domain names are also allowed and placed under the <b><i>not_dns</i></b> name space.  The concept of packages is derived from UML.

### XCN Class Names

XCN class names are based on UML and Java's class system.  Because of this, they MAY be fully qualified or relative.  However, this class system is abstracted away from UML and Java so that all languages MAY use it. See the subsequent sections for details.

##### Fully Qualified XCN Class Name

Fully qualified XCN class names must include their respective package names.

##### Relative XCN Class Name

### XCN Objects

### XCN Primitive Objects

There are four types of XCN Primitive Objects;

##### XCN Field Labels

Field Labels MUST start with a lower-case ASCII-7 / UTF-8 letter.  Field Labels MUST be less than 255 characters long.  Field Labels MUST be comprised of ASCII-7 / UTF-8 letters and numeric characters {i.e. 0-9}.  Field Labels SHOULD use [Camel Case](#camel-case).

##### XCN Booleans

Booleans MUST be comprised of one of the the following values {'t','f','true','false'}.

##### XCN Numbers

Numbers MUST be comprised of [Ten10b](#ten10b) text.

##### XCN Strings

Strings MUST start and end with double quotes '"'.  Similar to [JSON](#json-rfc-8259) the double quote MAY be escaped with a backslash, and the backslash MAY be escaped with a backslash.

```
"\"foo\"" // becomes → 'foo' in memory
"\\" // becomes → '\' in memory
```

### XCN Complex Objects

Complex Objects MUST NOT contain any data other than an OPTIONAL XCN Class Name.  XCN Objects MUST contain whitespace after the start-object left parentheses '(' character, or the subsequent XCL class name.  Complex Objects MUST close with a right parentheses ')'.  When Complex Objects contain child Complex Objects they are terminated with a closing tag, similar to [HTML](#html).  Complex Objects MAY be self terminating when the right parentheses is preceded by a slash '/', similar to [XML](#xml-wc3).

Fully qualified XCN class name: <b><i>not_dns.xcn.Object</i></b>

Examples;

```
// Without the Object class name
( /)

// With the Object relative class name
(Object /)

// With the Object fully qualified class name
(not_dns.xcn.Object /)

//
```

### XCN Arrays

Arrays are extensions of [Complex Objects](#xcn-complex-objects) which contain a pair of square brackets before the self terminating slash and right parentheses.  Inside of the square brackets '\[]', XCN Objects MAY be included.  XCN Arrays SHOULD be narrowed to Primitive and Complex.

##### Primitive Arrays

Primitive Arrays are a restriction of Arrays which MUST ONLY contain [Primitive Objects](#xcn-primitive-objects) or other Primitive Arrays.

```
// Example Primitive Array with the optional Array class name
(Array[ t 123 "abc" a]/)

// Example Primitive Array with-out the optional Array class name
([ t 123 "abc" a]/)

// Example Primitive Array of Primitive Arrays
(Array[ ([Array t 123 ]/) (Array[ "def" b]/)]/)


```

##### Complex Arrays

Inside of the square brackets '\[]', [Primitive Objects](#xcn-primitive-objects) or [Complex Objects](#xcn-complex-objects) MAY be included.

### XCN Line Segments

Line Segments MUST exist on a single line.  Line Segments MAY contain fields which MUST be [Primitive Objects](#xcn-primitive-objects) or [Primitive Arrays](#primitive-arrays).  XCN Line Segments MUST NOT contain fields comprised of [Complex Objects](#xcn-complex-objects).  XCN Line Segments extend from XCN Objects.

Fully qualified XCN class name: <b><i>not_dns.xcn.LineSegment</i></b>

Examples;

```
// Without the LineSegments class name
( foo="bar"/)

// With the LineSegment relative class name
(LineSegment bar=123/)

// With the Object fully qualified class name
(not_dns.xcn.LineSegment /)
```

##### XCN Trees

XCN Trees MAY contain simple fields and complex fields (i.e. other XCN Trees and Tables).   Conceptually, XCN Trees extend XCN Objects.

Fully qualified XCN class name: <b><i>not_dns.xcn.Tree</i></b>

##### XCN Tree Fields

[Complex Objects](#xcn-complex-objects) MAY contain one or more fields.  Fields MUST be comprised of a [Field Label](#xcn-field-labels) and an assigned field value.  Values are assigned to [Field Labels](#xcn-field-labels) with a equals symbol.  Field values MAY be other [Complex Objects](#xcn-complex-objects) or [Primitive Objects](#xcn-primitive-objects).  

Examples;

```
// Without the Tree class names
( foo=( car="bar"/)/)

// With the Tree relative class names
(Tree bar=(Tree car="foo"/)/)

// With the Tree fully qualified class names
(not_dns.xcn.Tree bar=(not_dns.xcn.Tree car="foo"/)/)

// Tree with internal table
( foo=
(Table rows=2 )
(Columns[ mo car ]/)
(Row[ "feb" 3 ]/)
(Row[ "mar" 36 ]/)
(/Table)
/)

```

##### XCN Tables

XCN Tables MUST NOT contain simple fields or complex fields (i.e. other XCN Trees and Tables).   Conceptually, XCN Trees extend XCN Objects.  The XCN Table start tag MUST be immediately followed by a UNIX Line Feed, then a XCN Columns tag and finally zero or more Row tags.  XCN Columns MUST consist of either fieldLables or strings.  The Table, Columns and Row tags all extend from [XCN Line Segments](#xcn-line-segments).

Example;

```
(Table id=3 rows=3 )
(Columns[ abc col2   "Col 3" treeRef ]/)
(Row[     t   "me"   123     1 ]/)
(Row[     f   "you"  345.3   2 ]/)
(Row[     f   "them"  99.0   5 ]/)
(/Table)
```


Notes: 
Linking to a child tree or table MAY be accomplished using XCN id references.
XCN Tables do NOT need to be pretty printed as they are done in these examples.
This was changed from the original brainstorming style in order to allow the optimization of adding the number of bytes in a particular row.

Example;

```
23(Table id=3 )
43(Columns[ abc col2   "Col 3" treeRef ]/)
37(Row[     t   "me"   123     1 ]/)
37(Row[     f   "you"  345.3   2 ]/)
37(Row[     f   "them"  99.0   5 ]/)
11(/Table)
```

# File Scan Optimizations

XCN data segments MAY be prefixed by one or more numbers separated by commas.  These numbers MAY be comprised of [Ten10B Integers](#ten10b) or [Ten64 Integers](#ten64).  Note that [Ten64 Integers](#ten64) are the most optimal, while [Ten10B Integers](#ten10b) split the difference between optimality and human readability.  The first number in the OPTIONAL sequence of numbers identifies the number of bytes in the data segment.  In table-style data structures, this identifies the number of bytes the row.

Additional integers may be added, separated by a comma, in order to identify bytes of interest.  For example, the primary key and identifier of a table may be a composite key with two column values.

```
23(Table id=3 )
30([ keyCol1 keyCol2 name ]/)
29,3,7([ 123 "xyz" "John"]/)
27,3,6([ 45 "n" "Sarah"]/)
25,3,5([ 7 "c" "Jane"]/)
11(/Table)
```

In the above example on line three, which starts with 29.  There are 29 bytes in the line.  The 'keyCol1' ID '123' starts at byte 3 (note the offset is zero-based, from the starting left parentheses).  The 'keyCol2' ID "xyz" starts at byte 7.  

Note that this kind of optimization significantly slows down writing of data but significantly speeds up reading of data.

TODO asymptotic analysis of writing and reading data

...

When using XCN, we are targeting O(log log n) seek time for optimized table formats, we believe this will be slightly better than the big O(log n) seek time found in Column Storage Formats [Row vs Column Formats](#row-vs-column-formats).

Also note this is somewhat similar to the [Neo4j](#neo4j) binary storage format, due to its use of line feeds indicating end of row data.

# Commentary

I was really hoping to leverage JSON in EJCN to do this kind of thing.  I feel like I'm re-inventing the wheel.  I think this sort of thing has been going on since well before I was born.

2026-04-21 Added XML/HTML style self closing tag, to distinguish between tags like Tables that SHOULD have closure with a ending tag.  Switched to a more LISP style syntax to make XCN look much different from XML.

Finally, I recommend you read this document at least twice to get the gist of it.

## Citations and Workflow Comments

Finally, note that most of the Github style Markdown to RFC style XML conversion, and citations were generated by [Gemini](#google-gemini).  Also, [Gemini Deep Research](#google-gemini-deep-research), found [ISO/IEC 10967 integer datatype section 5.1 / International Math Standard.](#math-international-standard), and other content that I didn't track.  Finally, note I dictated most of this paper using various voice-to-text AI software, which has given much of it a strangely verbal style. Feel free to reach out if you would like to correct, modify or add anything to this paper.

I often used the following prompt;

```
Find or create Academic/Footnote Markdown Chicago style and RFC XML style citations to
```

# Citations

## Citations and Workflow Comments

Finally, note that most of the Github style Markdown to RFC style XML conversion, and citations were generated by [Gemini](#google-gemini).  Also, [Gemini Deep Research](#google-gemini-deep-research), found [ISO/IEC 10967 integer datatype section 5.1 / International Math Standard.](#math-international-standard), and other content that I didn't track.  Finally, note I dictated most of this paper using various voice-to-text AI software, which has given much of it a strangely verbal style. Feel free to reach out if you would like to correct, modify or add anything to this paper.

I often used the following prompt;

```
Find or create Academic/Footnote Markdown style and RFC XML style citations to
```

##### Analog Wikipedia

Wikipedia contributors. "Analog device." *Wikipedia, The Free Encyclopedia*. Accessed April 5, 2026. <https://en.wikipedia.org/wiki/Analog_device>.

##### Apache HTTP Server Configuration Files

The Apache Software Foundation, "Configuration Files," Apache HTTP Server Version 2.4 Documentation, accessed April 9, 2026, https://httpd.apache.org/docs/current/configuring.html.

##### Base 10

Computers must frequently translate between binary data and human-readable base-10 (decimal) positional number systems.

Knuth, Donald E. *The Art of Computer Programming, Volume 2: Seminumerical Algorithms*. 3rd ed., Addison-Wesley, 1997, pp. 195. (Section 4.1: Positional Number Systems).

##### BigDecimal Mclaughlin

Because standard ECMAScript numbers are double-precision floats, JavaScript implementations should utilize an arbitrary-precision library such as `decimal.js` to ensure accurate base-10 calculations.

 Mclaughlin, Michael. "decimal.js - An arbitrary-precision Decimal type for JavaScript." *GitHub Pages*, https://mikemcl.github.io/decimal.js/. Accessed 6 Apr. 2026.

##### BigDecimal Java Oracle

For exact decimal arithmetic, such as financial calculations, the application must utilize a specialized data type rather than a standard floating-point number.

Oracle. "Class BigDecimal." *Java Platform, Standard Edition & Java Development Kit Version 25 API Specification*, Oracle Corporation, Sept. 2025. https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/math/BigDecimal.html. Accessed 6 Apr. 2026.

##### Binary Number Systems Wikipedia

Wikipedia contributors. "Binary number." *Wikipedia, The Free Encyclopedia*. Accessed April 5, 2026. <https://en.wikipedia.org/wiki/Binary_number>.

##### Camel Case

##### Compart

Compart. "Unicode Character “̅” (U+0305)." *Compart*. Accessed April 5, 2026. <https://www.compart.com/en/unicode/U+0305>.

##### Context Free Grammers

John E. Hopcroft, Rajeev Motwani, and Jeffrey D. Ullman, *Introduction to Automata Theory, Languages, and Computation*, 3rd ed. (Boston: Addison-Wesley, 2006).

https://dpvipracollege.ac.in/wp-content/uploads/2023/01/John-E.-Hopcroft-Rajeev-Motwani-Jeffrey-D.-Ullman-Introduction-to-Automata-Theory-Languages-and-Computations-Prentice-Hall-2006.pdf

##### Decimal Conversion C++

The C++ standard library provides a specific constant for this exact round-trip serialization guarantee called `max_digits10`.[^1]

[^1]: "std::numeric_limits\<T\>::max_digits10." *cppreference.com*, https://en.cppreference.com/w/cpp/types/numeric_limits/max_digits10. Accessed 5 Apr. 2026.

##### Decimals Wikipedia

Wikipedia contributors. "Decimal." *Wikipedia, The Free Encyclopedia*. Accessed April 5, 2026. <https://en.wikipedia.org/wiki/Decimal>.

##### Discrete System Wikipedia

Wikipedia contributors. "Discrete system." *Wikipedia, The Free Encyclopedia*. Accessed April 5, 2026. <https://en.wikipedia.org/wiki/Discrete_system>.

##### ECMAScript 2025

The engine parses the code according to the rules defined in the ECMAScript specification.[^1]

"ECMAScript® 2025 Language Specification." *Ecma International*, Standard ECMA-262, 16th ed., June 2025. https://tc39.es/ecma262/. Accessed 6 Apr. 2026.

##### EJCN (eXtensable JSON Classification Notation)

Adligo, "Extensible JSON Classification Notation (EJCN)," GitHub repository, 2026, https://github.com/adligo/ejcn.adligo.org.

##### Endianness Wikipedia

Wikipedia contributors. "Endianness." *Wikipedia, The Free Encyclopedia*. Accessed April 5, 2026. <https://en.wikipedia.org/wiki/Endianness>.

##### Fractions Wikipedia

Wikipedia contributors. "Fraction." *Wikipedia, The Free Encyclopedia*. Accessed April 5, 2026. <https://en.wikipedia.org/wiki/Fraction>.

##### Google Gemini

Emergent Mind. (2026, January 14). Google Gemini: Scalable multimodal models. Retrieved from https://www.emergentmind.com/topics/google-gemini

##### Google Gemini Deep Research

i10X. (2025, December 16). Google Gemini Deep Research: AI for complex tasks. Retrieved from https://i10x.ai/news/google-gemini-deep-research-analysis

##### Grammers Chomsky

Noam Chomsky, "On certain formal properties of grammars," *Information and Control* 2, no. 2 (1959): 137-167, https://doi.org/10.1016/S0019-9958(59)90362-6.

##### HTML

WHATWG, "HTML Standard," Web Hypertext Application Technology Working Group, accessed April 9, 2026, https://html.spec.whatwg.org/.

##### IEEE 754

Wikipedia contributors. "IEEE 754." *Wikipedia, The Free Encyclopedia*. Accessed April 5, 2026. <https://en.wikipedia.org/wiki/IEEE_754>.

##### Integers Wikipedia

Wikipedia contributors. "Integer." *Wikipedia, The Free Encyclopedia*. Accessed April 5, 2026. <https://en.wikipedia.org/wiki/Integer>.

##### Integral Wikipedia

Wikipedia contributors. "Integral." *Wikipedia, The Free Encyclopedia*. Accessed April 5, 2026. <https://en.wikipedia.org/wiki/Integral>.

##### Java Language Specification

Gosling, J., Joy, B., Steele, G., Bracha, G., Buckley, A., Smith, D., and Bierman, G. *The Java® Language Specification, Java SE 21 Edition*. Oracle America, Inc., September 2023. Accessed April 5, 2026. <https://docs.oracle.com/javase/specs/jls/se21/html/index.html>.

##### JSON RFC 8259

Many modern APIs use the JSON data interchange format.

Bray, Tim, editor. "The JavaScript Object Notation (JSON) Data Interchange Format." *Internet Engineering Task Force (IETF)*, RFC 8259, Dec. 2017. https://doi.org/10.17487/RFC8259. Accessed 5 Apr. 2026.

##### Line Wrapping Wikipedia

Wikipedia contributors. "Wrapping (text)." *Wikipedia, The Free Encyclopedia*. Accessed April 5, 2026. <https://en.wikipedia.org/wiki/Wrapping_(text)>.

##### Math International Standard

*Information technology — Language independent arithmetic — Part 1: Integer and floating point arithmetic*, ISO/IEC 10967-1:2012, Jul. 2012.

##### Modern Western Numeral System

Morgan, S. "Modern Western Numeral System." In *Text Encoded Base 64 Numbers (Ten64)*. Internet-Draft draft-morgan-ten64-00, April 1, 2026. <https://www.ietf.org/archive/id/draft-morgan-ten64-00.html#modern-western-numeral-system>.

##### Neo4j

Neo4j. (n.d.). *neo4j/neo4j: Graphs for Everyone* [Source code]. GitHub. Retrieved April 22, 2026, from https://github.com/neo4j/neo4j

##### Parquet

Apache Software Foundation. "Apache Parquet." Accessed April 22, 2026. https://parquet.apache.org/.

##### Positional Number Systems Wikipedia

Wikipedia contributors. "Positional notation." *Wikipedia, The Free Encyclopedia*. Accessed April 5, 2026. <https://en.wikipedia.org/wiki/Positional_notation>.

##### Row vs Column Formats

DataWithSantosh. (2024, June 11). *Row-Based Storage vs Column-Based Storage: A Beginner's Guide*. Medium. Retrieved April 22, 2026, from https://medium.com/@DataWithSantosh/row-based-storage-vs-column-based-storage-a-beginners-guide-6e91dbadb181

##### Ten10b

Adligo. "adligo/ten10b_v1.adligo.org: Text Encoded Base 10 Numbers (Ten10b) Version 1." GitHub. Accessed April 22, 2026. https://github.com/adligo/ten10b_v1.adligo.org.

##### Ten64

Morgan, S. "Text Encoded Base 64 Numbers (Ten64)." Internet-Draft draft-morgan-ten64-00, April 1, 2026. <https://www.ietf.org/archive/id/draft-morgan-ten64-00.html>.

##### Ten64 Commentary

Morgan, S. "Commentary." In *Text Encoded Base 64 Numbers (Ten64)*. Internet-Draft draft-morgan-ten64-00, April 1, 2026. <https://www.ietf.org/archive/id/draft-morgan-ten64-00.html#commentary>.

##### TypeScript

Microsoft. "TypeScript: Typed JavaScript at Any Scale." Accessed April 5, 2026. <https://www.typescriptlang.org/>.

##### UTF-8 RFC 3629

Yergeau, F. "UTF-8, a transformation format of ISO 10646." RFC 3629, STD 63, November 2003. <https://www.rfc-editor.org/info/rfc3629>.

The document must be structured according to the W3C XML specification.[^1]

##### XML WC3

Bray, Tim, et al., editors. "Extensible Markup Language (XML) 1.0 (Fifth Edition)." *World Wide Web Consortium (W3C)*, 26 Nov. 2008. https://www.w3.org/TR/xml/. Accessed 5 Apr. 2026.
