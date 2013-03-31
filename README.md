Handlebars.js-helpers-collection
================================

Handlebars.js helpers collection

### Overview

#### List helpers
* <a href="#First">Get first element of a list</a>
* <a href="#IterateSliced">Iterate over a given range</a>
* <a href="#commaSeparatedList">Print comma separated list</a>
* <a href="#JOIN">Join elements with character in between</a>
* <a href="#keyValue">Put/overrides value by given key</a>
* <a href="#each_with_key">Put/overrides all nested values by given key</a>


#### Conditions
* <a href="#IfEqual">If Equal</a>

#### Value formatting helpers
* <a href="#formatDate">Format Date</a>
* <a href="#encode">Encode characters</a>

#### Debugging and Logging
* <a href="#debugHelper">Debug Helper</a>

#### Managing Scope
* <a href="simpleInclude">Include of other Handlebar templates</a>


####<a name="keyValue">override Key-Value</a>
##### Description
Iterate over an object, setting 'key' and 'value' for each property in
the object.
#####Usage
{{#key_value obj}} Key: {{key}} // Value: {{value}} {{/key_value}}

```javascript
Handlebars.registerHelper("key_value", function(obj, fn) {
    var buffer = "",
        key;

    for (key in obj) {
        if (obj.hasOwnProperty(key)) {
            buffer += fn({key: key, value: obj[key]});
        }
    }

    return buffer;
});
```
by https://gist.github.com/strathmeyer


####<a name="each_with_key">Each with Key</a>
#####Description
Iterate over an object containing other objects. Each
inner object will be used in turn, with an added key ("myKey")
set to the value of the inner object's key in the container.
#####Usage
{{#each_with_key container key="myKey"}}...{{/each_with_key}}

```javascript
Handlebars.registerHelper("each_with_key", function(obj, fn) {
    var context,
        buffer = "",
        key,
        keyName = fn.hash.key;

    for (key in obj) {
        if (obj.hasOwnProperty(key)) {
            context = obj[key];

            if (keyName) {
                context[keyName] = key;
            }

            buffer += fn(context);
        }
    }

    return buffer;
});
```

by https://gist.github.com/strathmeyer

####<a name="First">First</a>
#####Description:
return the first item of a list only
#####Usage:
{{#first items}}{{name}}{{/first}}
```javascript
Handlebars.registerHelper('first', function(context, block) {
  return block(context[0]);
});
```


####<a name="IterateSliced">Iterate sliced</a>
#####Description
Iterate over a specific portion of a list.
#####Usage:
{{#slice items offset="1" limit="5"}}{{name}}{{/slice}} : items 1 thru 6
{{#slice items limit="10"}}{{name}}{{/slice}} : items 0 thru 9
{{#slice items offset="3"}}{{name}}{{/slice}} : items 3 thru context.length
defaults are offset=0, limit=5
#####Todo: combine parameters into single string like python or ruby slice ("start:length" or "start,length")

```javascript
Handlebars.registerHelper('slice', function(context, block) {
  var ret = "",
      offset = parseInt(block.hash.offset) || 0,
      limit = parseInt(block.hash.limit) || 5,
      i = (offset < context.length) ? offset : 0,
      j = ((limit + offset) < context.length) ? (limit + offset) : context.length;

  for(i,j; i<j; i++) {
    ret += block(context[i]);
  }

  return ret;
});
```


####<a name="commaSeparatedList">Print comma separated list</a>
#####Description
return a comma-serperated list from an iterable object
##### Usage:
{{#toSentance tags}}{{name}}{{/toSentance}}

```javascript
Handlebars.registerHelper('toSentance', function(context, block) {
  var ret = "";
  for(var i=0, j=context.length; i<j; i++) {
    ret = ret + block(context[i]);
    if (i<j-1) {
      ret = ret + ", ";
    };
  }
  return ret;
});
```

###<a name="formatDate">Format Date</a>
##### Description
format an ISO date using Moment.js
http://momentjs.com/
#####Usage:
moment(Date("2011-07-18T15:50:52")).format("MMMM YYYY")
{{dateFormat creation_date format="MMMM YYYY"}}
```javascript
Handlebars.registerHelper('dateFormat', function(context, block) {
  if (window.moment) {
    var f = block.hash.format || "MMM Do, YYYY";
    return moment(Date(context)).format(f);
  }else{
    return context;   //  moment plugin not available. return data as is.
  };
});
```

###<a name="debugHelper">Debug Helper</a>
#####Description

#####Usage
{{debug}} or {{debug someValue}}

```javascript
Handlebars.registerHelper("debug", function(optionalValue) {
  console.log("\nCurrent Context");
  console.log("====================");
  console.log(this);

  if (arguments.length > 1) {
    console.log("Value");
    console.log("====================");
    console.log(optionalValue);
  }
});
```

from: @commondream (http://thinkvitamin.com/code/handlebars-js-part-3-tips-and-tricks/)



###<a name="encode">encode</a>
#####Description
Runs a string through encodeURIComponent, returning the result. This is very useful when constructing URLs.
#####Usage:
{{uc page_id}}
```javascript
Handlebars.registerHelper('uc', function (str) {
    return encodeURIComponent(str);
});
```

```javascript
_isFunction = function(obj) {
    return !!(obj && obj.constructor && obj.call && obj.apply);
};

var head = function (arr, fn) {
    if (_isFunction(fn)) {
        // block helper
        return fn(arr[0]);
    }
    else {
        return arr[0];
    }
};

Handlebars.registerHelper('first', head);
Handlebars.registerHelper('head', head);

Handlebars.registerHelper('tail', function (arr, fn) {
    if (_isFunction(fn)) {
        // block helper
        var out = '';
        for (var i = 1, len = arr.length; i < len; i++) {
            out += fn(arr[i]);
        }
        return out;
    }
    else {
        return arr.slice(1);
    }
});
```


###<a name="simpleInclude">Include</a>
#####Description
Includes another handlebars template loaded using the optional build-steps described above. This is similar to using a partial, but it supports nested directories and file extensions:
Notice, I've used triple-mustaches to make sure the result of rendering that template isn't escaped. The current (not the root) context is used for rendering the included template.
by caolan https://github.com/caolan
##### Usage:
{{{include "path/to/template.html"}}}
```javascript
// TODO: add optional context argument?
Handlebars.registerHelper('include', function (name) {
    if (!exports.templates[name]) {
        throw new Error('Template Not Found: ' + name);
    }
    return exports.templates[name](this, {});
});
```


### IfEqual
#####Description:
Block-helper that executes the inner-block if the two arguments test as strict equal (===). This also supports else blocks.
#####Usage:
<ul>
  {{#ifequal current_page "home"}}
    <li><strong>home</strong></li>
  {{else}}
    <li><a href="/home">home</a></li>
  {{/ifequal}}
</ul>
```javascript
Handlebars.registerHelper('ifequal', function (val1, val2, fn, elseFn) {
    if (val1 === val2) {
        return fn();
    }
    else if (elseFn) {
        return elseFn();
    }
});
```

### JOIN
#####Description
Handlebars "join" block helper that supports arrays of objects or strings.
(implementation found here: https://github.com/wycats/handlebars.js/issues/133)
    
If "delimiter" is not speficified, then it defaults to ",".
You can use "start", and "end" to do a "slice" of the array.
Use with objects:
{{#join people delimiter=" and "}}{{name}}, {{age}}{{/join}}

Use with arrays:
{{join jobs delimiter=", " start="1" end="2"}}
```javascript
Handlebars.registerHelper('join', function(items, block) {
    var delimiter = block.hash.delimiter || ",", 
        start = start = block.hash.start || 0, 
        len = items ? items.length : 0,
        end = block.hash.end || len,
        out = "";
    
        if(end > len) end = len;
    
    if ('function' === typeof block) {
        for (i = start; i < end; i++) {
            if (i > start) out += delimiter;
            if('string' === typeof items[i])
                out += items[i];
            else
                out += block(items[i]);
        }
        return out;
    } else { 
        return [].concat(items).slice(start, end).join(delimiter);
    }
});
```
