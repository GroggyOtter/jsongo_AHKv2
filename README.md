# **jsongo** for AHKv2

Full JSON support for AHKv2 written natively in AHK.  
Closely mimics [Mozilla's JSON format](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON), including support for revivers, replacers, and spacers.

```
; Include jsongo to use it
#Include jsongo.v2.ahk

; Consider getting Peep() from my repository
; Peep() is a function that allows you to view the contents of any object
; https://github.com/GroggyOtter/PeepAHK
;#Include peep.v2.ahk

; Convert a JSON string to an AHK object
obj := jsongo.Parse(json [,reviver])
;Peep(obj)

; Convert an AHK object to a JSON string
json := Stringify(obj [,replacer ,spacer ,extract_all])
MsgBox(json)
```

## Contents

### 0) [Preface: Quick syntax clarification and Peep() recommendations](#0-quick-syntax-clarification-and-peep-recommendation)

### 1) [.Methods()](#1-methods-1)
  * **[Parse()](#parse)**
    * **[json [String]](#--json-string)**
    * **[reviver [Function] [optional]](#--reviver-function--optional)**
    * **[RETURN value](#--return--value)**

  * **[Stringify()](#stringify)**
    * **[obj [Map | Array | Object]](#--object-map--array--object)**
    * **[replacer [Function | Array] [optional]](#--replacer-function-optional)**
    * **[spacer [String | Number] [optional]](#--spacer-string--number-optional)**
    * **[extract_all [Bool] [optional]](#--extract_all-bool-optional)**
    * **[RETURN value](#--return--value-1)**

### 2) [.Properties](#2-properties-1)
  * [`.escape_slash`](#escape_slash)
  * [`.escape_backslash`](#escape_backslash)
  * [`.extract_objects`](#extract_objects)
  * [`.extract_all`](#extract_all)
  * [`.inline_arrays`](#inline_arrays)
  * [`.silent_error`](#silent_error)
  * [`.error_log`](#silent_error)

### 3) [Instructions/Guide + Examples](#3-instructionsguide--examples-1)
  * **[Using Parse(): JSON string to AHK object  
  => `obj := jsongo.Parse(json)`](#using-parse-to-convert-a-json-string-to-an-ahk-object)**
  * **[Parse Revivers: Access to key:value pairs  
  => `obj := jsongo.Parse(json, reviver)`](#parse-revivers-access-to-keyvalue-pairs)**
  * **[Using Stringify(): AHK object to JSON string  
  => `json := jsongo.Stringify(obj)`](#using-stringify-ahk-object-to-json-string)**
  * **[Stringify Replacers: Access to key:value pairs  
  => `json := jsongo.Stringify(obj, replacer)`](#stringify-replacers-access-to-keyvalue-pairs)**
  * **[Stringify Spacers: JSON string formatting option
  => `json := jsongo.Stringify(obj, , spacer)`](#stringify-spacers-json-string-formatting-option)**

### **4) [ChangeLog](#4-changelog-1)**
  * **[BETA](#beta---20230627)**

***

## 0) Quick syntax clarification and Peep() recommendation

#### Preface
I created a JSON library a while back for v1. Got it working correctly but never actually *finished* it.  
I've used it a handful of times and that's it.  
Coming across it the other day, I decided to compltely rewrite it for AHKv2.  
Here it is! I actually finished something!

#### Peep()
[`Peep()`](https://github.com/GroggyOtter/PeepAHK) is a function I wrote that allows you to view the contents of any object, including all nested objects.  
It then displays them in a tree structure very similar to how JSON looks.  
Since writing it, I find myself using `Peep()` consistently.  
I strongly recommend giving it a try.  
I'm not trying to self-promote. I'm mentioning it because the tool is pretty useful.  


#### `[]` Square Bracket Syntax

Any time you see `[]` square brackets around the parameters of a function or method, it indicates that those parameters are optional and can be omitted.  
This is standard convention with AHK and is used extensively in the AHK documents.  

Example using [`GetKeyState()`](https://www.autohotkey.com/docs/v2/lib/GetKeyState.htm) from the docs:  
```
IsDown := GetKeyState(KeyName [,Mode])
```

In that example, `KeyName` is *required*, but `Mode` is optional.

I wanted to add this for the coders who may not realize what the meaning of `[]` square brackets are in AHK syntax.

#### "Why call it jsongo?"

It stands for JSONGroggyOtter.  
jsongo is a short class name.  
It's identifiable.  
And the big reason is it leaves the `json` namespace open for use so you don't accidentally try to overwrite a class named `json` when you use that a variable or property name.

## 1) .Methods()  
[`Back to Contents`](#contents)  

***
### Parse()
Used to parse a string of JSON text into an AHK object.  
```
obj := jsongo.Parse(json [,reviver:=''])`
```

#### - `json` [[String](https://www.autohotkey.com/docs/v2/Concepts.htm#strings)]  
String containing JSON text.

#### - `reviver` [[Function](https://www.autohotkey.com/docs/v2/misc/Functor.htm)]  **[optional]**  
Used to access all `key:value` pairs parsed from the JSON string before they're added to the AHK object.  
`reviver` needs to be a reference to a [Function()/FuncObject](https://www.autohotkey.com/docs/v2/misc/Functor.htm) or [Method](https://www.autohotkey.com/docs/v2/Language.htm#operators-for-objects).  
It requires at least 3 parameters so it can receive  the current `key`, `value`, and a special `remove` variable.  
The function/method is expected to return the original value, an altered value, or no value (`''` an empty string).  
Alternatively, the `remove` variable from parameter 3 can be returned which instructs the parser to discard the current `key:value` pair and not add it to the object.

If no return value is sent, the value is replaced with `''` an empty string.  
If the `remove` variable provided in parameter 3 is returned, the parser is instructed to ignore that key:value pair completely.  

Revivers act very similarly to replacer functions/methods.  

#### - `RETURN => value`  
The return type is dependant on the JSON string provided.  
Possible return types: [Map](https://www.autohotkey.com/docs/v2/lib/Map.htm), [Array](https://www.autohotkey.com/docs/v2/lib/Array.htm), [String](https://www.autohotkey.com/docs/v2/Concepts.htm#strings), [Number](https://www.autohotkey.com/docs/v2/Concepts.htm#numbers)  
If an error occurs, the script will throw an error notifying you what happened, what was expected, and what was recieved.  
JSON syntax errors should also include a [clip of exactly where in the string the `>>>error<<<` happened](https://i.imgur.com/Nm4JRLj.png).  

If the [`silent_error` property](#silent_error) is true, no error is thrown, an empty string is returned, and the [`error_log` property](#error_log) is set to the error information.  
Remember that `.error_log` is cleared at the start of each `Parse()` and `Stringify()`.
 
***
### Stringify()
Used to convert an AHK object into a JSON string.  
```
json := Stringify(obj [,replacer:='' ,spacer:='' ,extract_all:=0])
```

#### - `obj` [[Map](https://www.autohotkey.com/docs/v2/lib/Map.htm) | [Array](https://www.autohotkey.com/docs/v2/lib/Array.htm) | [Object](https://www.autohotkey.com/docs/v2/lib/Object.htm)]  

JSON is about transmitting data, not creating structures and prototypes.  
By default, **jsongo** respects the core data ideology behind the JSON data structure and only accepts [Maps](https://www.autohotkey.com/docs/v2/lib/Map.htm), [Arrays](https://www.autohotkey.com/docs/v2/lib/Array.htm), [Strings](https://www.autohotkey.com/docs/v2/Concepts.htm#strings), and [Numbers](https://www.autohotkey.com/docs/v2/Concepts.htm#numbers).  
AHK has no true `true`, `false`, or `null` data types so they are converted to `1`, `0`, and `''` an empty string (respectively).  

[Literal Objects](https://www.autohotkey.com/docs/v2/Objects.htm#object-literal) are accepted if the [`.extract_objects` property](#extract_objects) is set to true.  
[All object types](https://www.autohotkey.com/docs/v2/ObjList.htm) are accepted if the [`.extract_all` property](#extract_all) or [`extract_all` parameter](#--extract_all-bool-optional) are set to true.  
Object types are exported in a `{key:value}` format, where the property name is used as the key.  
Key names must be a [String](https://www.autohotkey.com/docs/v2/Concepts.htm#strings) or an error will be thrown.  

#### - `replacer` [[Function](https://www.autohotkey.com/docs/v2/misc/Functor.htm)] **[optional]**  
Used to access all `key:value` pairs extracted from the AHK object before they're added to the JSON string.  
A `replacer` can be either a [Function()/FuncObject](https://www.autohotkey.com/docs/v2/misc/Functor.htm)/[Method()](https://www.autohotkey.com/docs/v2/Language.htm#operators-for-objects) or an [Array](https://www.autohotkey.com/docs/v2/lib/Array.htm).  

If a [Function()/FuncObject](https://www.autohotkey.com/docs/v2/misc/Functor.htm)/[Method()](https://www.autohotkey.com/docs/v2/Language.htm#operators-for-objects) reference is used, it requires at least 3 parameters so it can receive the current `key`, `value`, and a special `remove` variable.  
The function/method is expected to return the original value, an altered value, or no value (`''` an empty string).  
In addition, the `remove` variable from parameter 3 can be returned which instructs the parser to discard the current `key:value` pair and not add it to the string.

Replacer functions/methods act very similarly to revivers.  

If an [Array](https://www.autohotkey.com/docs/v2/lib/Array.htm) is used, the elements of the array represent a list of key names that should be removed.  
The key from every `key:value` pair is checked against that list and if a match is found, that key:value pair is completely discarded and the parser goes onto the next item.  

#### - `spacer` [[String](https://www.autohotkey.com/docs/v2/Concepts.htm#strings) | [Number](https://www.autohotkey.com/docs/v2/Concepts.htm#numbers)] **[optional]**  
Spacers are a way to influence the formatting of a JSON string.  
If `spacer` is a number, it indicates that many spaces should be used for each level of indentation.  
If `spacer` is a string, that character set is used for each level of indentation.  
Any set of characters can be used for a string `spacer`, however, if anything other than valid whitespace characters `[spaces, tabs, linefeeds, carriage returns]` are used, the JSON string will no longer be valid and cannot be converted back into an object (without additional formatting to remove the special spacer indentation characters).  
Using invalid whitespace characters should be reserved for human consumption.  

#### - `extract_all` [[bool](https://www.autohotkey.com/docs/v2/Concepts.htm#boolean)] **[optional]**   
Setting this to true is the same as setting the [`extract_all` property](#extract_all) to true.  
This provides on-demand object extraction without having to toggle the property on and off.  

* `true` => When using Stringify(), all [Object types](https://www.autohotkey.com/docs/v2/lib/Object.htm) will be processed and exported in `{key:value}` format.  
* `false` => **[Default]** Script will throw an error when encountering any object that's not an [Array](https://www.autohotkey.com/docs/v2/lib/Array.htm) or [Map](https://www.autohotkey.com/docs/v2/lib/Map.htm).  

As a warning, when extracting objects, it is possible for two objects to reference each other.  
X points to Y which points back to X which points back to Y, causing an infinite loop that eventually fills up the stack.  
This is known as a [Circular Reference](https://www.autohotkey.com/docs/v2/Objects.htm#Usage_Freeing_Objects).  
While this library does try to accommodate users wanting to do object extraction (even though it's not technically part of JSON's intended usage), it does not protect against circular references and will throw an error if one is encountered.  


#### - `RETURN => value`  
Returns a JSON string  
If an error occurs, the script will throw an error notifying you what happened, what was expected, and what was recieved.  

If the [`silent_error` property](#silent_error) is true, no error is thrown, an empty string is returned, and the [`error_log` property](#error_log) is set to the error information.  

Remember that `.error_log` is cleared at the start of each `Parse()` and `Stringify()`.


## 2) .Properties
[`Back to Contents`](#contents)  

***
### `.escape_slash`  
The `/` forward slash (aka solidus) is the only character in JSON that can be optionally escaped.  
This property allows you to set your preference on whether forward slashes should be escaped.  

* `true` => **[Default]** Forward slashes are escaped during Stringify().  
  Example: `{"URL":"https:\/\/github.com\/GroggyOtter\/jsongo_AHKv2\/"}`
* `false` => Forward slashes remain unescaped during Stringify().  
  Example: `{"URL":"https://github.com/GroggyOtter/jsongo_AHKv2/"}`

***
### `.escape_backslash`  
The `\` backslash (aka reverse solidus) can be escaped in two ways:  
Escape character: `\\`  
Unicode escaping: `\u005C`  
This property allows you to set your preference on which escaping to use.  

* `true` => **[Default]** Backslashes use escape character `\\`
* `false` => Backslashes use unicode escaping `\u005c`

***
### `.extract_objects`  
By default, **jsongo** will only work with [Arrays](https://www.autohotkey.com/docs/v2/lib/Array.htm) and [Maps](https://www.autohotkey.com/docs/v2/lib/Map.htm).  
This property allows **jsongo** to attempt and extract the properties and values from [Literal Objects](https://www.autohotkey.com/docs/v2/Objects.htm#object-literal).  

* `true` => When using Stringify(), all [Literal Objects](https://www.autohotkey.com/docs/v2/lib/Object.htm) will be processed and exported in `{key:value}` format.  
* `false` => **[Default]** Script will throw an error when encountering a literal object  
  
As a warning, when extracting objects, it is possible for two objects to reference each other.  
X points to Y which points back to X which points back to Y, causing an infinite loop that eventually fills up the stack.  
This is known as a [Circular Reference](https://www.autohotkey.com/docs/v2/Objects.htm#Usage_Freeing_Objects).  
While this library does try to accommodate users wanting to do object extraction (even though it's not technically part of JSON's intended usage), it does not protect against circular references and will throw an error if one is encountered.  

***
### `.extract_all`
By default, **jsongo** will only work with [Arrays](https://www.autohotkey.com/docs/v2/lib/Array.htm) and [Maps](https://www.autohotkey.com/docs/v2/lib/Map.htm).  
This property allows **jsongo** to attempt and extract the properties and values from [All object types](https://www.autohotkey.com/docs/v2/ObjList.htm).  

* `true` => When using Stringify(), all [Object types](https://www.autohotkey.com/docs/v2/lib/Object.htm) will be processed and exported in `{key:value}` format.  
* `false` => **[Default]** Script will throw an error when encountering any object that's not an [Array](https://www.autohotkey.com/docs/v2/lib/Array.htm) or [Map](https://www.autohotkey.com/docs/v2/lib/Map.htm).  

As a warning, when extracting objects, it is possible for two objects to reference each other.  
X points to Y which points back to X which points back to Y, causing an infinite loop that eventually fills up the stack.  
This is known as a [Circular Reference](https://www.autohotkey.com/docs/v2/Objects.htm#Usage_Freeing_Objects).  
While this library does try to accommodate users wanting to do object extraction (even though it's not technically part of JSON's intended usage), it does not protect against circular references and will throw an error if one is encountered.  

***
### `.inline_arrays`
This property allows the user to set a preference of putting all array values on the same line.  
This option only applies to arrays that do not contain other arrays or objects.  

* `true` => When using Stringify(), arrays containing only primitives ([Strings](https://www.autohotkey.com/docs/v2/Concepts.htm#strings) and/or [Numbers](https://www.autohotkey.com/docs/v2/Concepts.htm#numbers)) will be dispalyed on 1 line.  
* `false` => **[Default]** All array elements will be displayed on a separate line.  

Example:  
```
; jsongo.inline_arrays := true
["Cat", "Dog", "Emu"]

; jsongo.inline_arrays := false
[
    "Cat",
    "Dog",
    "Emu"
]

; jsongo.inline_arrays := true
[
    "String",
    3.14,
    [1,2,3]
]
```

***
### `.silent_error`
Added for automation purposes.  
This property gives user the ability to suppress all errors.  
Instead, when an error occurs, an empty string is returned and the [`.error_log` property](#error_log) is set to the error message that was supposed to be displayed.  

* `true` => No longer throws errors, returns an empty string on failure, and sets [`.error_log` property](#error_log) to error message.  
* `false` => **[Default]** On failure, the script will throw a normal pop-up error.

***
### `.error_log`  
If the [`.silent_error` property](#silent_error) is set to true and an error occurs, no pop-up message will be displayed.  
Instead, the error message is assigned to this property.  
This allows the user to verify an error actually occurred and that the returned empty string wasn't a valid return value.  


## 3) Instructions/Guide + Examples
[`Back to Contents`](#contents)  

***
### Using Parse() to convert a JSON string to an AHK object

The `Parse()` method is used to convert a string of JSON data into a usable AHK object.  

```
#Include jsongo.v2.ahk
json := '{"Array":["a","b","c"]}'
obj := jsongo.Parse(json)
for index, value in obj['Array']
    MsgBox(index ':' value)
```

Example using my own personal JSON test file (a much more complex JSON string):  

```
; JSON string
json := '{"key_00_JSON_start": "First item of the file","key_01_string": "String","key_02_made_by": "TheGroggyOtter","key_03_array_true_false_null": [true, false, null],"key_04_object_all_number_types":{"zero":{"pos_zero":0,"neg_zero":-0},"integer":{"pos_integer":186282,"neg_integer":-1000000000000066600000000000001},"decimal":{"pos_decimal":3.14159,"neg_decimal":-1.618,"pos_zero_decimal":0.57721,"neg_zero_decimal":-0.0},"exponent":{"pos_int_no_sign_E":43252003274489856E3,"neg_int_no_sign_E":-19e1,"neg_dec_pos_e":-0.271828e+1,"pos_dec_neg_e":6.626068e-34,"neg_dec_neg_E":-420000000000.0E-10,"pos_dec_pos_E":0.08675309E+8},"~numbers_used":"light, belphegor, pi, golden, e. con, rubiks, 3 stooge total, e. num, p. con, uni_ans, jenny"},"key_05_string_stuff":{"ALPHA UPPER": "ABCDEFGHIJKLMNOPQRSTUVWXYZ","alpha lower": "abcdefghijklmnopqrstuvwxyz","Specials Chars": "!@#$%^&*()_+-=[]{}<>,./?;`':","Digits - Value": "0123456789","0123456789": "Digits As Key","key_case_check": "lowercase key","KEY_CASE_CHECK": "UPPERCASE KEY","JSON text string": "{\"array with str, num, t, f, n\": [\"string\", 3.14, true, false, null]}","Quotation Marks": "&#34; \u0022 %22 0x22 034 &#x22;","Code comment symbols": "\/\/ \/* *\/ <!-- --> `; # rem `' C {- =begin =end -- --[[ ]] % (* \"\"\" ```","Escape Characters":{"ESC_01 Quotation Mark": "\"","ESC_02 Backslash/Reverse Solidus": "\\","ESC_03 Slash/Solidus (Not a mandatory escape)": "\/ and / work","ESC_04 Backspace": "\b","ESC_05 Formfeed": "\f","ESC_06 Linefeed": "\n","ESC_07 Carriage Return": "\r","ESC_08 Horizontal Tab": "\t","ESC_09 Unicode": "\u00AF\\_(\u30C4)_\/\u00AF \u0CA0_\u0CA0","ESC_10 All": "\\\/\"\b\f\n\r\t\u0033","ESC_11 \u2660\u2665\u2663\u2666": "Encodes in key"}},"key_06_nested_arrays_[matrix]":[[1, 0, 0, 0, 0],[0, 1, 0, 0, 0],[0, 0, 1, 0, 0],[0, 0, 0, 1, 0],[0, 0, 0, 0, 1]],"key_07_nested_objects":{"Person object":{"name":{"first":"Groggy","last":"Otter","user":"GroggyOtter"},"job": "Professional Geek","favorites":{"color":["Black","White"],"food":["Pizza","Cheeseburger","Steak"]}},"vehicle object":{"make": "Subaru","model": "WRX STI","color":{"Primary": "Black","Secondary": "Red"},"transmission": {"Manual": true,"Automatic": false}}},"key_08_empty_things":{"empty object": {},"empty object with whitespace": {      },"empty array": [],"empty array with whitespace": [		],"empty string": "","empty promises": true},"key_09_spacing":{"1_json_whitespace":["Spaces for padding. Linefeed at end.","Tabs for padding. Carriage return at end.","Tabs/space mix for padding. CR+LF at end."],"2_compact_text":[1,2,3,4,"a","b","c","d"],"3_formatted_text" :[0,1,2,3,4,0,1,2,3,0,1,2,0,1,0 ],"4_scattered_text" :["This ","is","considered","valid ","spacing. ","JSON ","only ","cares ","about ","whitespace ","inside ","of ","strings."]},"key_10_JSON_end": "Last item of the file"}'

obj := jsongo.Parse(json)
; peep(obj)   ; If you're using my Peep() function
MsgBox(obj['key_07_nested_objects']['Person object']['name']['user'])
ExitApp()
```

***

### Parse Revivers: Access to `key:value` pairs

The purpose of the `reviver` is to give the user access to every `key:value` pair before they are added to the AHK object.  
This gives the user the option to keep, alter, or delete the value as well as the option to completely omit the `key:value` pair.

A `reviver` must be a [Function()/FuncObject](https://www.autohotkey.com/docs/v2/misc/Functor.htm) or [Method](https://www.autohotkey.com/docs/v2/Language.htm#operators-for-objects).  
It is passed 3 parameters: `key`, `value`, and a special `remove` variable.  

The `key` will be either a [String](https://www.autohotkey.com/docs/v2/Concepts.htm#strings) or a [Number](https://www.autohotkey.com/docs/v2/Concepts.htm#numbers) and can be checked with the [`Type()` function](https://www.autohotkey.com/docs/v2/lib/Type.htm).  
A [Number](https://www.autohotkey.com/docs/v2/Concepts.htm#numbers) key indicates an array index.  
A [String](https://www.autohotkey.com/docs/v2/Concepts.htm#strings) key indicates an object key.  

With this information, the user can build a funtion to alter/remove values in any way they choose.

Example of how a reviver function should be structured.  
```
; At least 3 parameters
reviver_function(key, value, remove) {
    ; Return the original value if you want it to remain unchanged
    return value
}
```

The 3rd parameter is the `remove` parameter.  
It's a special value that, when returned, instructs the parser to discard the current `key:value` pair.  

If you wanted to remove all numbers when importing the JSON string to an object, you'd use a reviver like this:

```
#Include jsongo.v2.ahk

json := '{"string":"some text", "integer":420, "float":4.20, "string2":"more text"}'
obj := jsongo.Parse(json, remove_numbers)
obj.Default := 'Key does not exist'
;Peep(obj) ; If you've included peep.v2.ahk
MsgBox('string2: ' obj['string2'] '`ninteger: ' obj['integer'])
ExitApp()

; Function to remove numbers
remove_numbers(key, value, remove) {
    ; When a value is a number (meaning Integer or Float)
    if (value is Number)
        ; Remove that key:value pair
        return remove
    ; Otherwise, return the original value for use
    return value
}

```

It's worth noting that you can have any amount of parameters and they can be optional parameters.  
The only requirement is there must be a place for all 3 parameters to go or AHK will throw an error (obviously).

This is a valid reviver:
```
reviver_function(key:='', *) {
    if (key == 'temp_info')
        return remove
    else return value
}
```

You wouldn't be able to do much with it because you don't have access to the `value` or the `remove` variables as they're absorbed by the `*` parameter tampon.  
Yes, that is what I call it.  
In actuality, it's a [variadiac parameter](https://www.autohotkey.com/docs/v2/Functions.htm#Variadic) that can take in any amount of parameters and puts them all into an array.  
With no value assigned to the array, the array is never created and all values fizzle. (Meaning they are discarded.)

Here's another reviver example that formats phone numbers to a `(###) ###-####` format.  
It also corrects email addresses from `@ahk.com` to `@autohotkey.com`:
```
json := 
(
'[
    {
    "name": "anon",
    "email": "anon@ahk.com",
    "phone": "5555143474"
    },
    {
    "name": "edc",
    "email": "edc@ahk.com",
    "phone": "5555032675"
    },
    {
    "name": "plank",
    "email": "plank@ahk.com",
    "phone": "5555103222"
    }
]'
)

obj := jsongo.Parse(json, format_phone_num)
; Phone numbers are now formatted as (###) ###-####
str := ''
for k, v in obj
    str .= 'name: ' v['name'] '`nemail: ' v['email'] '`nphone: ' v['phone'] '`n`n'
MsgBox(str)

format_phone_num(key, value, remove) {
    if (key == "phone")
        return RegExReplace(value, '(\d{3})(\d{3})(\d{4})', '($1) $2-$3')
    if (key == 'email')
        return StrReplace(value, '@ahk.com', '@autohotkey.com')
    return value
}
```

***
### Using Stringify(): AHK object to JSON string

The `Stringify()` method is used to convert an AHK object into a JSON string.  
    
```
#Include jsongo.v2.ahk
obj := Map('array',[1,2,3])
json := jsongo.Stringify(obj)
MsgBox(json)
```

By default, object can be a [Map](https://www.autohotkey.com/docs/v2/lib/Map.htm), [Array](https://www.autohotkey.com/docs/v2/lib/Array.htm), [String](https://www.autohotkey.com/docs/v2/Concepts.htm#strings), or [Number](https://www.autohotkey.com/docs/v2/Concepts.htm#numbers).  

JSON is about transmitting data, not creating structures and prototypes.  
That's why **jsongo** respects the core data ideology behind the JSON data structure and defaults to only accepting Arrays, Maps, Strings, and Numbers by default.  
Everything else will throw an error.  

[Literal Objects](https://www.autohotkey.com/docs/v2/Objects.htm#object-literal) are accepted if the [`.extract_objects` property](#extract_objects) is set to true.  

[All object types](https://www.autohotkey.com/docs/v2/ObjList.htm) are accepted if the [`.extract_all` property](#extract_all) or [`extract_all` parameter](#--extract_all-bool-optional) are set to true.  

Object types are exported in a `{key:value}` format, where the property name is used as the key.  
Key names must be a [String](https://www.autohotkey.com/docs/v2/Concepts.htm#strings) or an error will be thrown.  


***
### Stringify Replacers: Access to `key:value` pairs

The purpose of the `replacer` is to give the user access to each `key:value` pair before they're added to the JSON string.  
This gives the user the option to keep, alter, or delete the value as well as the option to completely omit the `key:value` pair.

A `replacer` can be either a [Function()/FuncObject](https://www.autohotkey.com/docs/v2/misc/Functor.htm)/[Method()](https://www.autohotkey.com/docs/v2/Language.htm#operators-for-objects) or an [Array](https://www.autohotkey.com/docs/v2/lib/Array.htm).  

If `replacer` is a function, it is passed 3 parameters: `key`, `value`, and a special `remove` variable.  
The user can then decide if they want to do with the value or `key:value` pair.

Example of a `replacer` that redacts the name from any key called `secret_identity`:

```
#Include jsongo.v2.ahk
obj := [Map('first_name','Bruce' ,'last_name','Wayne' ,'secret_identity','Batman')
        ,Map('first_name','Peter' ,'last_name','Parker' ,'secret_identity','Spider-Man')
        ,Map('first_name','Steve' ,'last_name','Gray' ,'secret_identity','Lexikos')]
json := jsongo.Stringify(obj, remove_hidden_identity, '`t')

MsgBox(json)

remove_hidden_identity(key, value, remove) {
    if (key == 'secret_identity')
        ; Tells parser to discard this key:value pair
        return RegExReplace(value, '.', '#')
    ; If no matches, return original value
    return value
}
```

If `replacer` is an [Array](https://www.autohotkey.com/docs/v2/lib/Array.htm), the items of the array are treated as a list of forbidden key names.  
They key of each `key:value` pair is checked against each item of the replacer array.  
If a match is ever made, that `key:value` pair is discarded and not included in the JSON string.  
Example of an array replacer:
```
#Include jsongo.v2.ahk
; Starting JSON
obj := Map('1_array_tfn', [true, false, '']
            ,'2_object_num', Map('zero',0
                            ,'-zero',-0
                            ,'int',7
                            ,'-float',-3.14
                            ,'exp',170e-2
                            ,'phone_num',5558675309)
            ,'3_escapes', ['\','/','"','`b','`f','`n','`r','`t']
            ,'4_unicode', '¯\_(ツ)_/¯')

arr := ['2_object_num', '3_escapes']
json := jsongo.Stringify(obj, arr, '`t')
MsgBox('2_object_num and 3_escapes do not appear in the JSON text output:`n`n' json)
```

### Stringify Spacers: JSON string formatting option

The purpose of a `spacer` is to format the JSON string.  
`spacer` can be a [String](https://www.autohotkey.com/docs/v2/Concepts.htm#strings) or a [Number](https://www.autohotkey.com/docs/v2/Concepts.htm#numbers).  

If `spacer` is a number, it indicates that many spaces should be used for each level of indentation.  

```
#Include jsongo.v2.ahk
obj := Map('array',[1,2,3])
json := jsongo.Stringify(obj, , 4)
; Exports with 4 spaces for each level of indentation:
; {
;     "array": [
;         1,
;         2,
;         3
;     ]
; }

```

If `spacer` is a String, the string provided is used for each level of indentation.  
You are allowed to use any character in the `spacer` string, however, using any characters other than valid whitespace `Space, Tab, Linefeed, Carriage Return` will cause the exported JSON file to no longer be a valid JSON string.  
It cannot be imported back into an object without removing the custom characters from the indentation.  
This can easily be achieved with something like [`RegExReplace()`](https://www.autohotkey.com/docs/v2/lib/RegExReplace.htm))

Try to reserve invalid characters for JSON intended for human consumption.  
Personally, I like using `'|   '` as a spacer because it makes a connecting line between elements which makes it easier to read.  
```
#Include jsongo.v2.ahk
obj := Map('matrix',[[1,2,3]
                   ,[4,5,6]
                   ,[7,8,9]])
json := jsongo.Stringify(obj, , '|    ')
MsgBox(json)

; Displays as:
;  {
;  |    "matrix": [
;  |    |    [
;  |    |    |    1,
;  |    |    |    2,
;  |    |    |    3
;  |    |    ],
;  |    |    [
;  |    |    |    4,
;  |    |    |    5,
;  |    |    |    6
;  |    |    ],
;  |    |    [
;  |    |    |    7,
;  |    |    |    8,
;  |    |    |    9
;  |    |    ]
;  |    ]
;  }


```

If `spacer` is `''` an empty string or the `spacer` parameter is omitted, no formatting is used and the exported JSON string will be exported as one single line of text.  
Remember, actual Linefeeds (new lines) that occur in JSON strings are always encoded as `\u000A` or `\n` as required by the JSON standard.  
This is why everything can export as one line of text when no formatting is applied.  
This is also the most efficient way of exporting and importing JSON data as it has the fewest characters to parse through, making it he the fastest.

```
#Include jsongo.v2.ahk
obj := Map('array1',[[1,2,3]
                    ,[4,5,6]
                    ,[7,8,9]]
            ,'array2',[['a','b','c']
                      ,['d','e','f']
                      ,['g','h','i']])
json := jsongo.Stringify(obj)
MsgBox(json)
; Exports as:
; {"array1":[[1,2,3],[4,5,6],[7,8,9]],"array2":[["a","b","c"],["d","e","f"],["g","h","i"]]}
```

## 4) ChangeLog
[`Back to Contents`](#contents)  

### 1.0 - 20230731
* Officially updated to v1.0
* Updated documents to JSDoc comments
  * This benefits editors like VS Code that utilize JSDoc tags.
  * Intellisense tooltip info should now be much more robust.

### Beta - 20230627  
* Initial release of jsongo



