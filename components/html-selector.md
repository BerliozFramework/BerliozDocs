---
breadcrumb:
  - Components
  - HTML Selector
keywords:
  - html
  - css
  - selector
  - jquery
  - dom
  - query
---

# HTML Selector

> ℹ️ **Note**: The HTML selector is part of the **Berlioz** ecosystem but is available as a standalone package:
> [`berlioz/html-selector`](https://github.com/BerliozFramework/HtmlSelector).
> You can find it on [Packagist](https://packagist.org/packages/berlioz/html-selector).
> You can use it independently of the framework, in any PHP application.

**Berlioz HTML Selector** is a PHP library to do queries on HTML files with CSS selectors like *jQuery* on DOM.

## Load HTML

You can easily load an HTML string or file with the static function `HtmlSelector::query()`. For files, use second
parameter `contentsIsFile` of method.

```php
$htmlSelector = new \Berlioz\HtmlSelector\HtmlSelector();

$query = $htmlSelector->query('<html><body>...</body></html>');
$query = $htmlSelector->query('path-of-my-file/file.html', contentsIsFile: true);
$query = $htmlSelector->query(new SimpleXMLElement(/*...*/));
```

## Load from `ResponseInterface`

`HtmlSelector::queryFromResponse()` permit loading html of a response body.

```php
$htmlSelector = new \Berlioz\HtmlSelector\HtmlSelector();

/** @var \Psr\Http\Message\ResponseInterface $response */
$query = $htmlSelector->queryFromResponse($response);
```

## Do a query

It's very simple to query an HTML string with a selector like *jQuery*.

```php
/** @var \Berlioz\HtmlSelector\Query\Query $query */
$query = $query->find('body > .wrapper h2');
$query = $query->filter(':first');
```

## Selectors

### CSS Simple selectors

- **type**: selection of elements with their type.
- **#id**: selection of an element with it's ID.
- **.class**: selection of elements with their class.
- Attributes selections.
    - **[attribute]**: with attribute 'attribute'.
    - **[attribute=foo]**: value of attribute equals to 'foo'.
    - **[attribute^=foo]**: value of attribute starts with 'foo'.
    - **[attribute$=foo]**: value of attribute ends with 'foo'.
    - **[attribute*=foo]**: value of attribute contains 'foo'.
    - **[attribute!=foo]**: value of attribute different of 'foo'.
    - **[attribute~=foo]**: value of attribute contains word 'foo'.
    - **[attribute|=foo]**: value of attribute contains prefix 'foo'.

### CSS Ascendants, descendants, multiples

- ***selector* *selector*** or ***selector* >> *selector***: all descendant selector.
- ***selector* > *selector***: direct descendant selector (only children).
- ***selector* ~ *selector***: siblings selector.
- ***selector*, *selector***: multiple selectors.

### CSS Pseudo Classes

- **:any(selector, selector)**: only elements given in arguments.
- **:any-link**: only elements of type `<a>`, `<area>` and `<link>`, with `[href]` attribute.
- **:blank**: only elements without child, and no text (except spaces).
- **:checked**: only elements with attribute `[checked]`.
- **:dir**: only elements with directional text given (default: ltr).
- **:disabled**: only elements of type `<button>`, `<input>`, `<optgroup>`, `<select>` or `<textarea>` with `[disabled]`
  attribute.
- **:empty**: only elements without child.
- **:enabled**: only elements of type `<button>`, `<input>`, `<optgroup>`, `<option>`, `<select>`, `<textarea>`
  , `<menuitem>` or `<fieldset>` without `[disabled]` attribute.
- **:first**: only first result of complete selection.
- **:first-child**: only firsts children in their parents.
- **:first-of-type**: only firsts type in their parents.
- **:has(selector, selector)**: only elements who valid child selector.
- **:lang(x)**: only elements with attribute `[lang]` prefixed by or equals to given value.
- **:last-child**: only lasts in their parents.
- **:last-of-type**: only lasts type in their parents.
- **:not(selector, selector)**: filter 'not'.
- **:nth-child()**: *n* elements in selector result.
- **:nth-last-child()**: *n* elements in selector result, start at end of list.
- **:nth-of-type()**: *n* elements of given type in selector result.
- **:nth-last-of-type()**: *n* elements of given type in selector result, start at end of list.
- **:only-child**: only elements who are only child in the parent.
- **:only-of-type**: only elements who are only type child in the parent.
- **:optional()**: only input elements without `[required]` attribute.
- **:read-only()**: only elements that the user cannot edit.
- **:read-write()**: only elements with editable property.
- **:required()**: only elements with `[required]` attribute.
- **:root()**: get root element.

### Additional CSS Pseudo Classes (not in CSS specifications) from jQuery library

- **:button**: only elements of type `<button>` without attribute value `[type=submit]` or `<input type="button">`.
- **:checkbox**: only elements with attribute `[type=checkbox]`.
- **:contains(x)**: only elements who contain text given.
- **:eq(x)**: only result with index given (index start to 0).
- **:even**: only even results in selection.
- **:file**: only elements with attribute `[type=file]`.
- **:gt(x)**: only result with an index greater than index given (index start to 0).
- **:gte**: only result with an index greater than or equal to index given (index start to 0).
- **:header**: only elements of heading, like `<h1>`, `<h2>`...
- **:image**: only elements with attribute `[type=image]`.
- **:input**: only elements of type `<input>`, `<textarea>`, `<select>` or `<button>`.
- **:last**: only last result of complete selection.
- **:lt**: only result with index leather than index given (index start to 0).
- **:lte**: only result with index leather than or equal to index given (index start to 0).
- **:odd**: only odd results in selection.
- **:parent**: only elements with one child or more.
- **:password**: only elements with attribute `[type=password]`.
- **:radio**: only elements with attribute `[type=radio]`.
- **:reset**: only elements with attribute `[type=reset]`.
- **:selected**: only elements of type `<option>` with attribute `[selected]`.
- **:submit**: only elements of type `<button>` or `<input>` with attribute `[type=submit]`.
- **:text**: only elements of type `<input>` with attribute `[type=text]` or without `[type]` attribute.

### Additional CSS Pseudo Classes (not in CSS specifications)

- **:count(x)**: only elements who are x children in the parent, used in **:has(selector)** pseudo class.

### Full example of selectors

```
select > option:selected
div#myId.class1.class2[name1=value1][name2=value2]:even:first
```

## Functions

### Default functions

Some default functions are available in Query object to interact with results. The functions should have the same result
as their counterparts on jQuery.

- **attr(name)**: get attribute value
- **attr(name, value)**: set attribute value
- **children()**: get children of elements in result.
- **count()**: count the number of elements in query result.
- **data(nameOfData)**: get data value (name is with camelCase syntax without the 'data-' prefix).
- **filter(selector)**: filter elements in result.
- **find(selector)**: find selector in elements in result.
- **get(i)**: get DOM element in result.
- **hasClass(class_name)**: know if least one of element in result have given classes.
- **addClass(class_names)**: add space-separated classes to all elements.
- **removeClass(class_names)**: remove space-separated classes from all elements.
- **toggleClass(class_names, test)**: toggle classes on all elements. Optional `test` parameter forces add/remove.
- **html()**: get html of first element in result.
- **index(selector)**: get the index of given selector in result elements.
- **is(selector)**: know if selector valid the least one element in result.
- **isset(i)**: return boolean to know if an element key exists in result.
- **next(selector)**: get next element after each element in result.
- **nextAll(selector)**: get all next elements after each element in result.
- **not(selector)**: filter elements in result.
- **parent()**: get direct parent of current result of selecting.
- **parents(selector)**: get all parents of current result of selecting.
- **prev(selector)**: get prev element after each element in result.
- **prevAll(selector)**: get all prev elements after each element in result.
- **prop(name)**: get property boolean value of an attribute, used for example for `disabled` attribute.
- **prop(name, value)**: set property boolean value of an attribute, used for example for `disabled` attribute.
- **remove(selector)**: remove elements from the DOM. Optional selector filters which elements to remove.
- **map(callback)**: iterate elements and collect return values. Callback receives `(Query $query, int $key)`.
- **serialize()**: serialize input values of a form. Return a string.
- **serializeArray()**: serialize input values of a form. Return an array.
- **text()**: get text of each element concatenated.
- **val()**: get value of a form element.

## Extension system

You can register custom CSS pseudo-classes by implementing `ExtensionInterface`:

```php
use Berlioz\HtmlSelector\Extension\ExtensionInterface;
use Berlioz\HtmlSelector\PseudoClass\PseudoClass;
use Berlioz\HtmlSelector\PseudoClass\PseudoClassInterface;

class MyExtension implements ExtensionInterface
{
    public function getPseudoClasses(): array
    {
        return [
            new PseudoClass('visible', fn(string $xpath) => $xpath . '[not(@hidden)]'),
        ];
    }
}

$htmlSelector = new \Berlioz\HtmlSelector\HtmlSelector();
$htmlSelector->addExtension(new MyExtension());
```

Two built-in extensions are registered by default: `CssExtension` (standard CSS pseudo-classes) and `QueryExtension`
(jQuery-inspired pseudo-classes like `:button`, `:contains`, `:eq`, etc.).
