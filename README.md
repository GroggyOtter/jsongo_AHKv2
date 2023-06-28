# jsongo for AHKv2

Full JSON support for AHKv2 written natively in AHK.  
Closely mimics Mozilla's JSON format, using `Parse()` and `Stringify()`.  
Includes parse reviver, stringify replacer, and stringify spacer options.  

## Contents

### Methods:
&nbsp;&nbsp;&nbsp;&nbsp;**`.Parse()`**  
&nbsp;&nbsp;&nbsp;&nbsp;**`.Stringify()`**  

### Properties
&nbsp;&nbsp;&nbsp;&nbsp;**`.extract_objects`**  
&nbsp;&nbsp;&nbsp;&nbsp;**`.extract_all`**  
&nbsp;&nbsp;&nbsp;&nbsp;**`.escape_slash`**  
&nbsp;&nbsp;&nbsp;&nbsp;**`.inline_arrays`**  
&nbsp;&nbsp;&nbsp;&nbsp;**`.silent_error`**  

### Examples
&nbsp;&nbsp;&nbsp;&nbsp;**JSON String to AHK Object Example**  
&nbsp;&nbsp;&nbsp;&nbsp;**AHK Object to JSON String Example**  
&nbsp;&nbsp;&nbsp;&nbsp;**Parse Reviver Example**  
&nbsp;&nbsp;&nbsp;&nbsp;**Stringify Replacer Example**  
&nbsp;&nbsp;&nbsp;&nbsp;**Stringify Spacer Example**  

### ChangeLog

***

## Use:

To convert a JSON string into an AHK object, Parse() it.

```object := jsongo.Parse(jsonString)```

To convert an AHK object into a JSON string, stringify it

```jsonString := jsongo.Parse(object)```

***
## Methods:

#### `.Parse(string, [reviver := ''])`  
Used to parse a string of JSON text into an AHK object.
* `string`
 String containing JSON text.
* `reviver`
 Reference to a function, method, or funcobject that accepts at least 3 parameters.  
 Parameters will accept the current `key`, `value`, and a special `remove` variable (respectively).  
 Function must always return one of the following: Original value, modified value, or the `remove` variable from parameter 3.  
 These function similarly to to Stringify() replacers.  
 See `Examples` section for more information on using revivers.  
* RETURN value  
 Returns an object or a primitive depending on the JSON input.  
 Possible return types: Map, Array, String, Number  
 If an error occurs, the script with throw an error  
 If the `silent_error` property is true, no error is thrown, an empty string is returned.  
 
####`.Stringify(object, [replacer := '', spacer := '', extract_all := 0])`
Used to convert an AHK object into a JSON string.  
* `object`  
 Accepted input: `Map`, `Array`, `String`, `Number`  
 `Literal Objects` are accepted if the `extract_objects` property is set to true. They will be exracted as a map.  
 `All` object types are accepted if either the `extract_all` property or parameter are set to true.  
* `replacer` [Optional]  
 Reference to a function, method, or funcobject that has at least 3 parameters.  
 Parameters will accept the current `key`, `value`, and a special `remove` variable (respectively).  
 Function must always return one of the following: Original value, modified value, or the `remove` variable from parameter 3.  
 These function similarly to to Parse() rerivers.  
 See `Examples` section for more information on using replacers.  
* `spacer` [Optional]  
 Spacers are a way to format the JSON string output.  
 A string of valid whitespace characters to use for each level of indenting.
 The only valid characters are: [spaces, tabs, linefeeds, carriage returns]
 You are allowed to use invalid characters, but doing so should be reserved for human consumption only as the resulting JSON will be invalid and unparseable afterward.  
* RETURN value  
 Returns a valid JSON string
 Possible return types: Map, Array, String, Number
 If an error occurs, the script with throw an error
 If the `silent_error` property is true, no error is thrown, an empty string is returned.


## Properties

### `extract_objects`
### `extract_all`
### `escape_slash`

## Examples

### JSON String to AHK Object Example  
### AHK Object to JSON String Example  
### Parse Reviver Example  
### Stringify Replacer Example  
### Stringify Spacer Example  

## ChangeLog
***
### Beta - 20230627  
* Initial release of jsongo

