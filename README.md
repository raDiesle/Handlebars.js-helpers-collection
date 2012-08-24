Handlebars.js-helpers-collection
================================

Handlebars.js helpers collection

### overview

* <a href="#First">First</a>
* <a href="#IterateSliced">Iterate_Sliced</a>
* <a href="#commaSeparatedList">Print comma separated list</a>
* <a href="#formatDate">Format Date</a>
* <a href="#debugHelper">Debug Helper</a>


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
##### Upsage:
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
usage: {{debug}} or {{debug someValue}}
from: @commondream (http://thinkvitamin.com/code/handlebars-js-part-3-tips-and-tricks/)
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