# Parse XML in the Browser

You can easily parse XML in a browser using `window.DOMParser`.

```js
if (window.DOMParser)
  {
    parser = new DOMParser();
    xmlDoc = parser.parseFromString(txt, "text/xml");
  }
else // Internet Explorer
  {
    xmlDoc = new ActiveXObject("Microsoft.XMLDOM");
    xmlDoc.async = false;
    xmlDoc.loadXML(txt);
  }
```

And get specific values from the nodes like this:

```js
//Gets house address number
xmlDoc.getElementsByTagName("streetNumber")[0].childNodes[0].nodeValue;

//Gets Street name
xmlDoc.getElementsByTagName("street")[0].childNodes[0].nodeValue;

//Gets Postal Code
xmlDoc.getElementsByTagName("postalcode")[0].childNodes[0].nodeValue;
```

http://stackoverflow.com/questions/17604071/parse-xml-using-javascript
