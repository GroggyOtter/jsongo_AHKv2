# jsongo for AHKv2

Full JSON support for AHKv2 written natively in AHK.  
Closely mimics Mozilla's JSON format, including the optional parameters.  
```
; JSON string to AHK object
object := Parse(json [,reviver])

; AHK object to JSON string
json := Stringify(object [,replacer ,spacer ,extract_all])
```

## Contents

### **[Methods](#methods-1)**
  * **[`.Parse()`](#parsejson-reviver--)**
  * **[`.Stringify()`](#stringifyobject-replacer---spacer---extract_all--0)**
  * **[`.Pretty()`]()**

### **[Properties](#properties-1)**
  * **[`.escape_slash`](#escape_slash)**
  * **[`.escape_backslash`](#escape_backslash)**
  * **[`.extract_all`](#extract_all)**
  * **[`.extract_objects`](#extract_objects)**
  * **[`.inline_arrays`](#inline_arrays)**
  * **[`.silent_error`](#silent_error)**

### **[Use](#use-1)**
  * **`Parse()`**
  * **`Stringify()`**
  * **`Revivers`**
  * **`Replacers`**
  * **`Spacers`**


### **[Examples](#examples-1)**
 * **[JSON String to AHK Object Example](#json-string-to-ahk-object-example)**
 * **[AHK Object to JSON String Example](#ahk-object-to-json-string-example)**
 * **[Parse Reviver Example](#parse-reviver-example)**
 * **[Stringify Replacer Example](#stringify-replacer-example)**
 * **[Stringify Spacer Example](#stringify-spacer-example)**

### **[ChangeLog](#changelog)**

***

## Methods
[`Back to Contents`](#contents)  

### `.Parse(json, [reviver := ''])`

Used to parse a string of JSON text into an AHK object.  

* `json` [String]  
 String containing JSON text.

* `reviver` [Callback] **[OPTIONAL]**  
 Used to access all key:value pairs of the object before the value is assigned to the objct.  
 Reference to a function, method, or funcobject that accepts at least 3 parameters.  
 Parameters will accept the current `key`, `value`, and a special `remove` variable (respectively).  
 Function must always return one of the following: Original value, modified value, or the `remove` variable from parameter 3.  
 These function similarly to to Stringify() replacers.  
 See `Examples` section for more information on using revivers.

* `RETURN => value`  
 Returns an object or a primitive depending on the JSON input.  
 Possible return types: Map, Array, String, Number  
 If an error occurs, the script with throw an error.  
 If the `silent_error` property is true, no error is thrown and an empty string is returned.
 
### `.Stringify(object, [replacer := '', spacer := '', extract_all := 0])`  
Used to convert an AHK object into a JSON string.  
* `object` [Map | Array]  
  Accepted input: `Map`, `Array`, `String`, `Number`  
  `Literal Objects` are accepted if the `extract_objects` property is set to true. They will be exracted as a map.  
  `All` object types are accepted if either the `extract_all` property or parameter are set to true.  
* `replacer` [Callback] **[OPTIONAL]**  
  Used to access all key:value pairs of the object before being exported to JSON string.  
  Reference to a function, method, or funcobject that has at least 3 parameters.  
  Parameters will accept the current `key`, `value`, and a special `remove` variable (respectively).  
  Function must always return one of the following: Original value, modified value, or the `remove` variable from parameter 3.  
  These function similarly to to Parse() rerivers.  
  See `Examples` section for more information on using replacers.  
* `spacer` [String | Number] **[OPTIONAL]**  
  Spacers are a way to format the JSON string output.  
  If spacer is a number, that many spaces will be used for intdentation.  
  If a string is used, the charcter set provided will be used for each level of indentation.  
  Valid whitespace [spaces, tabs, linefeeds, carriage returns] are the only valid charcters for indenting.  
  You are allowed to use other characters, however doing so should be reserved for human consumption only as the resulting JSON string is no longer valid and can't be parsed back to an object.  
* `RETURN => value`  
  Returns a valid JSON string  
  Possible return types: Map, Array, String, Number  
  If an error occurs, the script with throw an error  
  If the `silent_error` property is true, no error is thrown, an empty string is returned.  

## Properties
[`Back to Contents`](#contents)  

### `.escape_slash`  
The forward slashe (solidus) is the only character in JSON that can be optionally escaped.  
* `true` => **[Default]** Forward slashes are escaped during Stringify().  
  Example: `{"URL":"https:\/\/github.com\/GroggyOtter\/jsongo_AHKv2\/"}`
* `false` => Forward slashes export without the escape character.  
  Example: `{"URL":"https://github.com/GroggyOtter/jsongo_AHKv2/"}`

### `.escape_backslash`  
Backslashes can be escaped two ways: Escape character `\\` and unicode escaping `\u005C`.
* `true` => **[Default]** Backslashes will be escaped `\\`
* `false` => Backslashes are escaped using unicode `\u005c`

### `.extract_all`
* `true` => When exporting an AHK object, any object derived from the base Object will be processed and exported as a map.  
  Script will loop through any object's `OwnProps()` and save each property as a key:value pair.  
* `false` => **[Default]** Script will throw an error when encountering any object that's not an Array or Map.  

### `.extract_objects`  
* `true` => When exporting an AHK object, literal objects will be processed and exported as a map.
  Script will loop through object's `OwnProps()` and save each property as a key:value pair.
* `false` => **[Default]** Script will throw an error when encountering a literal object
  
### `.inline_arrays`
* `true` => When exporting an AHK object, arrays containing only primitives (strings, integers, floats) will stay dispalyed on 1 line.  
  Example:  
  `["Cat", "Dog", "Emu"]`
* `false` => **[Default]** When exporting an AHK object, all array elements display on their own line:  
  Example:
  ```
  [
      "Cat",
      "Dog",
      "Emu"
  ]
  ```

### `.silent_error`
Added for automation purposes.  
Prevents having to worry about error popups.

* `true` => No longer displays an error and returns an empty string on failure.  
* `false` => **[Default]** Script will throw an error explaining why and where the error was thrown.

## Use:  
[`Back to Contents`](#contents)  

### JSON text to Object: `Parse(json)`

The `Parse()` method is used to turn a JSON string into an AHK object.  

```
object := jsongo.Parse(jsonString)
```

### Parse Revivers: `Parse(json, reviver)`

The reviver should be a reference to a function or method.  
It requires at least 3 parameters to receive the key name, value name, and a remove variable.  

Example of how a reviver function should be structured.  
```
; At least 3 parameters
reviver_function(key, value, remove) {
	; Always return the original value
	return value
}
```

It's worth noting that You can have any amount of parameters and they can be optional.  
The only requirement is that there's a place for all 3 parameter to be passed into. 

If you were going to blank out all values with a certain key name, the only parameter you need is the `key` parameter.  
This would be a valid reviver:

```
reviver_function(key, value, *) {
	if (key == 'temp_info')
	    return ''
	return value
}
```

`remove` is absorbed by the parameter tampon `*`. Yes, that is what I call it.  
It's actually a variadiac parameter that takes in any amount of parameters and puts them into an array.  
With no value assigned the array, an array is never created and all the values fizzle.  
Meaning they're discarded.

### AHK objec to JSON text `Stringify(object)`

The `Stringify()` method is used to turn an AHK object into a JSON string.  

```
json := jsongo.Stringify(object)
```

Object can be a Map, Array, String, Integer, or Float.  
By default, jsongo respects JSON's data structure and doesn't accept any other objects.  
JSON is about transmitting data, not creating structures and prototypes in multiple languages.  

This library comes with options some might want.  
`extract_objects` causes AHK to additionally accpet literal objects.  
It attempts to loop through all object properites and export them in associative array (map) format.  
`extract_all` causes AHK to accpet any object type.  
It attempts to loop through all object properites and export them in associative array (map) format.

### Stringify()

### Stringify()

```
jsonString := jsongo.Parse(object)
```



## Examples
[`Back to Contents`](#contents)  

### JSON String to AHK Object Example  

```
; Include jsongo
#Include jsongo.v2.ahk
; If you have Peep(), include it to see the full object:
; you can get Peep() from: https://github.com/GroggyOtter/PeepAHK
;#Included Peep.v2.ahk

; Valid JSON string
json := '{"1_array_tfn": [true, false, null],"2_object_num": {"zero":0,"-zero":-0,"int":42,"-float":-3.14,"exp":170e-2,"expnosign":-8.100e10,"phone_num":5558675309},"3_escapes": ["\\","\/","\"","\b","\f","\n","\r","\t"],"4_unicode": "\u00AF\\\u005F(\u30C4)\u005F\/\u00AF"}'

; Parse JSON string into an AHK object
obj := jsongo.Parse(json)

; Verify data is correct
MsgBox(obj['2_object_num']['int'])
; If you have Peep() installed:
Peep(obj)

ExitApp()
```

### AHK Object to JSON String Example  

```
; Include jsongo
#Include jsongo.v2.ahk
; If you have Peep(), include it to see the full object:
; you can get Peep() from: https://github.com/GroggyOtter/PeepAHK
;#Included Peep.v2.ahk

; Valid AHK Object
obj := Map("1_array_tfn",[true, false, '']
          ,"2_object_num", Map("zero",0
                              ,"int",42
                              ,"-float",-3.14
                              ,"exp",170e-2
                              ,"phone_num",5558675309)
          ,"3_escapes", ["\","/","`"","`b","`f","`n","`r","`t"]
          ,"¯\_(ツ)_/¯", "Unicode Key")

; Stringify object into a JSON string
json := jsongo.Stringify(obj)

; Verify data is correct
MsgBox(json['2_object_num']['int'])
; If you have Peep() installed:
; Peep(json)

ExitApp()
```

### Parse Reviver Example  
### Stringify Replacer Example  
### Stringify Spacer Example  

## ChangeLog
[`Back to Contents`](#contents)  

### Beta - 20230627  
* Initial release of jsongo
