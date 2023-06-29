# jsongo for AHKv2

Full JSON support for AHKv2 written natively in AHK.  
Closely mimics [Mozilla's JSON format](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON), including support for revivers, replacers, and spacers.

```
; JSON string to AHK object
obj := jsongo.Parse(json [,reviver])

; AHK object to JSON string
json := Stringify(obj [,replacer ,spacer ,extract_all])
```

## Contents

### **1) [.Methods()](#1-methods-1)**
  * **[`.Parse(json [,reviver])`](#parsejson-reviver)**
    * **[json [String]](#--json-string)**
    * **[reviver [Function] [optional]](#--reviver-function--optional)**
  * **[`.Stringify(object [,replacer ,spacer ,extract_all])`](#stringifyobject-replacer-spacer-extract_all0)**
    * **[object [Map | Array | Object]](#--object-map--array--object)**
    * **[replacer [Function | Array] [optional]](#--replacer-function-optional)**
    * **[spacer [String | Number] [optional]](#--spacer-string--number-optional)**
    * **[extract_all [Bool] [optional]](#--extract_all-bool-optional)**

### **2) [.Properties](#2-properties-1)**
  * **[`.escape_slash`](#escape_slash)**
  * **[`.escape_backslash`](#escape_backslash)**
  * **[`.extract_all`](#extract_all)**
  * **[`.extract_objects`](#extract_objects)**
  * **[`.inline_arrays`](#inline_arrays)**
  * **[`.silent_error`](#silent_error)**
  * **[`.error_log`](#silent_error)**

### **3) [Instructions/Guide + Examples](#3-instructionsguide--examples-1)**
  * **[Using Parse(): JSON string to AHK object  
  => `obj := jsongo.Parse(json)`](#using-parse-json-string-to-ahk-object)**
  * **[Parse Revivers: Access to key:value pairs  
  => `obj := jsongo.Parse(json, reviver)`](#parse-revivers-access-to-keyvalue-pairs)**
  * **[Using Stringify(): AHK object to JSON string  
  => `json := jsongo.Stringify(obj)`](#using-stringify-ahk-object-to-json-string)**
  * **[Stringify Replacers: Access to key:value pairs  
  => `json := jsongo.Stringify(obj, replacer)`](#stringify-replacers-access-to-keyvalue-pairs)**
  * **[Stringify Spacers: JSON sting formatting options  
  => `json := jsongo.Stringify(obj, , spacer)`](#stringify-spacers-json-sting-formatting-options)**

### **4) [ChangeLog](#4-changelog-1)**
  * **[BETA](#beta---20230627)**

***

## Quick syntax clarification

Any time you see `[]` square brackets around function or method parameters, it indicates that those parameters are optional and can be omitted.  

Example using [`GetKeyState()`](https://www.autohotkey.com/docs/v2/lib/GetKeyState.htm) from the docs:  
```
IsDown := GetKeyState(KeyName [,Mode])
```

`KeyName` is required, but `Mode` is optional.

Just adding that for the coders who didn't know what `[]` square brackets around parameters meant.

## 1).Methods()  
[`Back to Contents`](#contents)  

***
### `.Parse(json [,reviver:=''])`
***
Used to parse a string of JSON text into an AHK object.  

#### - `json` [[String](https://www.autohotkey.com/docs/v2/Concepts.htm#strings)]  
String containing JSON text.

#### - `reviver` [[Function](https://www.autohotkey.com/docs/v2/misc/Functor.htm)]  **[optional]**  
Used to access all `key: value pairs of the object before the value is assigned to the object.  
`reviver` must be a reference to a [Function()/FuncObject](https://www.autohotkey.com/docs/v2/misc/Functor.htm) or [Method](https://www.autohotkey.com/docs/v2/Language.htm#operators-for-objects) that accepts at least 3 parameters.  
Parameters will accept the current `key`, `value`, and a special `remove` variable (respectively).  
Function must always return one of the following: Original value, modified value, or the `remove` variable from parameter 3.  
If nothing is returned, an empty string is used as the return value.  
See [Guide Section](#json-text-to-object-parsejson) for more information on using revivers.

#### - `RETURN => value`  
Possible return types: [Map](https://www.autohotkey.com/docs/v2/lib/Map.htm), [Array](https://www.autohotkey.com/docs/v2/lib/Array.htm), [String](https://www.autohotkey.com/docs/v2/Concepts.htm#strings), [Number](https://www.autohotkey.com/docs/v2/Concepts.htm#numbers)  
If an error occurs, the script will throw an error  
If the [`silent_error` property](#silent_error) is true, no error is thrown, an empty string is returned, and the [`error_log` property](#error_log) is set to the error information.  
Remember that `.error_log` is cleared at the start of each `Parse()`.
 
***
### `.Stringify(object [,replacer:='' ,spacer:='' ,extract_all:=0])`  
***
Used to convert an AHK object into a JSON string.  
#### - `object` [[Map](https://www.autohotkey.com/docs/v2/lib/Map.htm) | [Array](https://www.autohotkey.com/docs/v2/lib/Array.htm) | [Object](https://www.autohotkey.com/docs/v2/lib/Object.htm)]  

JSON is about transmitting data, not creating structures and prototypes.  
By default, jsongo respects the core data ideology behind the JSON data structure and only accepts Arrays, Maps, Strings, and Numbers by default.  
`true`, `false`, and `null` are also accepted, however `true` converts to `1`, `false` converts to `0`, and `null` converts to `''` an empty string.  

[`Literal Objects`](https://www.autohotkey.com/docs/v2/Objects.htm#object-literal) are included and have their properties extracted in `{key:value format}` if the [`extract_objects property`](#extract_objects) is set to true.  

[`All object types`](https://www.autohotkey.com/docs/v2/ObjList.htm) are included and have their properties extracted in `{key:value format}` if [`extract_all property`](#extract_all) or [`extract_all parameter`](#--extract_all-bool-optional) are set to true.  

#### - `replacer` [[Function](https://www.autohotkey.com/docs/v2/misc/Functor.htm)] **[optional]**  
Used to access all `key:value` pairs of the object before being exported to JSON string.  
Reference to a [Function()/FuncObject](https://www.autohotkey.com/docs/v2/misc/Functor.htm) or [Method](https://www.autohotkey.com/docs/v2/Language.htm#operators-for-objects) that has at least 3 parameters.  
Parameters will accept the current `key`, `value`, and a special `remove` variable (respectively).  
Function must always return one of the following: Original value, modified value, or the `remove` variable from parameter 3.  
These function similarly to Parse() revivers.  

#### - `spacer` [[String](https://www.autohotkey.com/docs/v2/Concepts.htm#strings) | [Number](https://www.autohotkey.com/docs/v2/Concepts.htm#numbers)] **[optional]**  
Spacers are a way to format the JSON string output.  
If `spacer` is a number, that many spaces will be used for indentation.  
If `spacer` is a string, the character set provided will be used for each level of indentation.  
Valid whitespace [spaces, tabs, linefeeds, carriage returns] are the only valid characters for indenting.  
You are allowed to use other characters, however doing so should be reserved for human consumption only as the resulting JSON string is no longer valid and can't be parsed back to an object (though, I may add a reversal feature in the future...)

#### - `extract_all` [[bool](https://www.autohotkey.com/docs/v2/Concepts.htm#boolean)] **[optional]**   
Setting this to true is the same as setting the [`extract_all`](#extract_all) property to true.  

#### - `RETURN => value`  
Returns a JSON string  
If an error occurs, the script will throw an error  
If the `silent_error` property is true, no error is thrown, an empty string is returned, and the `error_log` property is set to the error information.
Remember that `.error_log` is cleared at the start of each `Stringify()`.



## 2) .Properties
[`Back to Contents`](#contents)  

***
### `.escape_slash`  
The forward slash (solidus) is the only character in JSON that can be optionally escaped.  
* `true` => **[Default]** Forward slashes are escaped during Stringify().  
  Example: `{"URL":"https:\/\/github.com\/GroggyOtter\/jsongo_AHKv2\/"}`
* `false` => Forward slashes export without the escape character.  
  Example: `{"URL":"https://github.com/GroggyOtter/jsongo_AHKv2/"}`

***
### `.escape_backslash`  
Backslashes (reverse solidus) can be escaped in two ways:  
Escape character `\\` and Unicode escaping `\u005C`.
* `true` => **[Default]** Backslashes will be escaped `\\`
* `false` => Backslashes are escaped using unicode `\u005c`

***
### `.extract_all`

* `true` => When exporting an AHK object to JSON, all valid [Object types](https://www.autohotkey.com/docs/v2/lib/Object.htm) will be processed and exported in `{key:value}` format.  
* `false` => **[Default]** Script will throw an error when encountering any object that's not an [Array](https://www.autohotkey.com/docs/v2/lib/Array.htm) or [Map](https://www.autohotkey.com/docs/v2/lib/Map.htm).  

As a warning, when extracting objects, it is possible for two objects to reference each other.  
X points to Y which points back to X which points back to Y, causing an infinite loop that eventually fills up the stack.  
This is known as a [Circular Reference](https://www.autohotkey.com/docs/v2/Objects.htm#Usage_Freeing_Objects).  
While this library does try to accommodate users wanting to do object extraction (even though it's not technically part of JSON's intended usage), it does not protect against circular references and will throw an error if one is encountered.  

***
### `.extract_objects`  
* `true` => When exporting an AHK object, [Literal Objects](https://www.autohotkey.com/docs/v2/Objects.htm#object-literal) will be processed and exported in `{key:value}` format.  
* `false` => **[Default]** Script will throw an error when encountering a literal object  
  
As a warning, when extracting objects, it is possible for two objects to reference each other.  
X points to Y which points back to X which points back to Y, causing an infinite loop that eventually fills up the stack.  
This is known as a [Circular Reference](https://www.autohotkey.com/docs/v2/Objects.htm#Usage_Freeing_Objects).  
While this library does try to accommodate users wanting to do object extraction (even though it's not technically part of JSON's intended usage), it does not protect against circular references and will throw an error if one is encountered.  

***
### `.inline_arrays`
* `true` => When exporting an AHK object, arrays containing only primitives ([Strings](https://www.autohotkey.com/docs/v2/Concepts.htm#strings)/[Numbers](https://www.autohotkey.com/docs/v2/Concepts.htm#numbers)) will be dispalyed on 1 line.  
* `false` => **[Default]** When exporting an AHK object, all array elements display on their own line:  
  Example:  
  ```
  ; true
  ["Cat", "Dog", "Emu"]
  
  ; false
  [
      "Cat",
      "Dog",
      "Emu"
  ]
  ```

***
### `.silent_error`
Added for automation purposes.  

* `true` => No longer throws errors.  
  On failure, an empty string is returned and the [`.error_log` property]() is set to the error info that would normally be displayed.  
* `false` => **[Default]** Script will throw errors when it encounters them.

***
### `.error_log`  
This property is set to an empty string before each Parse() and Stringify().  
If the `.silent_error` property is set to true and an error occurs, the error message is set to this property value.  
This can be used to verify an error actually occurred and that the intentional return value wasn't an empty string.



## 3) Instructions/Guide + Examples
[`Back to Contents`](#contents)  

***
### Using Parse(): JSON string to AHK object

The `Parse()` method is used to convert a JSON string into an AHK object.  

```
#Include jsongo.v2.ahk
json := '{"Array":["a","b","c"]}'
obj := jsongo.Parse(json)
for index, value in obj['Array']
	MsgBox(index ':' value)
```

And nested object example:
```
; JSON string
json := '{"1_array_tfn": [true, false, null],"2_object_num": {"zero":0,"-zero":-0,"int":7,"-float":-3.14,"exp":170e-2,"phone_num":5558675309},"3_escapes": ["\\","\/","\"","\b","\f","\n","\r","\t"],"4_unicode": "\u00AF\\\u005F(\u30C4)\u005F\/\u00AF"}'

obj := jsongo.Parse(json)
; peep(obj)   ; Reminder my Peep() script allows you to view any AHK object
MsgBox(obj['2_object_num']['-float'])
ExitApp()
```

***
### Parse Revivers: Access to key:value pairs

The purpose of the `reviver` is to give the user access to every `key:value` pair before they are added to the AHK object.  
This gives the user the option to keep, alter, or delete the value as well as the option to completely omit the key:value pair.

The `reviver` must be a function.  
It is passed 3 parameters: `key`, `value`, and a special `remove` variable.  

The `key` will be either a [String](https://www.autohotkey.com/docs/v2/Concepts.htm#strings) or a [Number](https://www.autohotkey.com/docs/v2/Concepts.htm#numbers) and can be checked with the [`Type() function`](https://www.autohotkey.com/docs/v2/lib/Type.htm).  
A `Number` key indicates an array index.  
A `String` key indicates an object key.  

With this information, the user can build a reviver to alter values in any way they choose.

Example of how a reviver function should be structured.  
```
; At least 3 parameters
reviver_function(key, value, remove) {
	; Return the original value if you want it to remain unchanged
	return value
}
```

The 3rd parameter, what I refer to as the `remove` parameter, is a special value that, when returned, instructs the parser to discard the current `key:value` pair.  

If you wanted to remove all `key:value` pairs that were labeled `temp_info`, you'd use a reviver like this:

```
; The JSON string contains a temp_info key
json := '{"keep_info_1":"data","temp_info":"you should not see this.","keep_info_2":"More data."}'
obj := jsongo.Parse(json, remove_temp_info)
str := ''
for key, value in obj
    str .= '`n' key ':' value
MsgBox(str)

remove_temp_info(key, value, remove) {
	; When a temp_info key is encountered
    if (key == 'temp_info')
        ; Remove it from the object
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

You wouldn't be able to do much with it other than remove everything the ability to remove things because the `remove` parameter is absorbed by the `*` parameter tampon.  
Yes, that is what I call it.  
In actuality, it's a [variadiac parameter](https://www.autohotkey.com/docs/v2/Functions.htm#Variadic) that can take in any amount of parameters and puts them all into an array.  
With no value assigned to the array, the array is never created and all values fizzle. (Meaning they are discarded.)

Example of a reviver function that formats phone numbers from a `5551234567` format to a `(555) 123-4567` format and correct email address from @ahk.com to @autohotkey.com:
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
That's why jsongo respects the core data ideology behind the JSON data structure and defaults to only accepting Arrays, Maps, Strings, and Numbers by default.  
Everything else will throw an error.  

[Literal objects](https://www.autohotkey.com/docs/v2/Objects.htm#object-literal) are included if the [`.extract_objects` property](#extract_objects) is set to true.  

[All object types](https://www.autohotkey.com/docs/v2/ObjList.htm) are included if [`.extract_all` property](#extract_all) or [`extract_all` parameter](#--extract_all-bool-optional) are set to true.  

In both cases, the object's `OwnProps()` is called and the data is exported as `{property_nam:value}` a map (JSON associative array).  

***
### Stringify Replacers: Access to key:value pairs

The purpose of the `replacer` is to give the user access to each `key:value` pair before they're added to the JSON string.  
This gives the user the option to keep, alter, or delete the value as well as the option to completely omit the key:value pair.

A `replacer` can be either a [Function](https://www.autohotkey.com/docs/v2/misc/Functor.htm)] or an [Array](https://www.autohotkey.com/docs/v2/lib/Array.htm).  

If `replacer` is a function, it is passed 3 parameters: `key`, `value`, and a special `remove` variable.  
The user can then decide what they want to do with the value.  
Keep, alter, delete, or remove the whole key:value pair from the string.  

Example of a `replacer` that removes social security numbers from an object before converting to JSON:

```
replacer(key, value, remove) {
    if (key == "ssn")
    	; Tells parser to discard this key:value pair
    	return remove
	; If no matches, return original value
	return value
}
```

If `replacer` is an array, the array is treated as a list of forbidden key names.  
If any key name matches something anything in the array, that key:value pair is discarded and not included in the JSON string.  
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

### Stringify Spacers: JSON string formatting options

`spacer` can be a String or a Number.  
The purpose of a spacer is to format the JSON output text into something more human-readable.  

If `spacer` is a number, it indicates that many spaces should be used for each level of indentation.  

```
obj := Map('array',[1,2,3])
json := jsongo.Stringify(obj, , 4)
; Exports as:
; {
;     "array": [
;         1,
;         2,
;         3
;     ]
; }

```

If `spacer` is a String, the string provided is used for each level of indentation.  
While you **can** use any character in the spacer string, using anything other than valid whitespace `Space, Tab, Linefeed, Carriage Return` will cause the exported JSON file to no longer be a valid JSON string and thus cannot be imported back into an object (unless the custom formatting is removed, which could be easily done with a [`RegExReplace()`](https://www.autohotkey.com/docs/v2/lib/RegExReplace.htm)).  

As long as the exported JSON is for human consumption, it's fine to use any characters you want.  
Personally, I like using `'|   '` as a spacer because it makes a connecting line between elements which makes it easier for humans to read.  
```
obj := Map('array',[[1,2,3],[4,5,6],[7,8,9]])
json := jsongo.Stringify(obj, , '|    ')
MsgBox(json)

; Displays as:
; "{
; |    "array":[
; |    |    [
; |    |    |    1,
; |    |    |    2,
; |    |    |    3
; |    |    ],
; |    |    [
; |    |    |    4,
; |    |    |    5,
; |    |    |    6
; |    |    ],
; |    |    [
; |    |    |    7,
; |    |    |    8,
; |    |    |    9
; |    |    ]
; |    ]
; }"
```

If `spacer` is `''` an empty string, or if the `spacer` parameter is omitted, no formatting is used and the exported JSON string will be exported as one single line of text.  
Remember, actual Linefeeds that occur in strings are always encoded as `\u000A` as required by the JSON standard.  
This is why everything exports as one line of text when no formatting is applied.
This is also the most efficient way of exporting and importing JSON data and should be used when humans never need to look at it.

```
obj := Map('array',[[1,2,3],[4,5,6],[7,8,9]])
json := jsongo.Stringify(obj)
MsgBox(json)
; Exports as:
; {"array":[[1,2,3],[4,5,6],[7,8,9]]}
```

## 4) ChangeLog
[`Back to Contents`](#contents)  

### Beta - 20230627  
* Initial release of jsongo

