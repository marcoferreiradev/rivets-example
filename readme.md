# How Do I Rivets

A buddy recently turned me on to [rivets js](http://www.rivetsjs.com/). Besides being bundled with the epic [ember js](http://emberjs.com/) framework, it's a sleek little stand alone lib. Rivets is a [templating system](http://en.wikipedia.org/wiki/JavaScript_templating), which in and of itself is not so revolutionary, except unlike handlebars or mustache it has built in two-way binding. This means that, assuming everything is configured properly, when you update your data object the DOM will auto-magically update, and when you update properly associated DOM elements the data model will update. A one and done, one stop shop, configure your page and don't worry about your view and your data falling out of sync again.

This tutorial is going to walk you through building a small mock-shopping app with a product list, a shopping bag and a bill calculator. This app will be for a candy shop.

### Hello Rivets

The most basic thing you could do in rivets is use it's handlebars-like syntax to populate parts of the dom with data. This is called "text content interpolation", but for our purposes it basically means defining where you want data to appear in the DOM by wrapping it in curly brackets: `{ somedata }`.

#### html

Make a new html file and write the below markup in it. Title this file "index.html".

```html
<!DOCTYPE html>
<html>
  <title>Candy Shop</title>
    
  <link rel="stylesheet" href="candy-shop.css">
  <script type="text/javascript" 
      src="http://cdn.rawgit.com/mikeric/rivets/master/dist/rivets.min.js"></script>
  <body>
    <div id="candy-shop">
      <h1>{ data.title }</h1>
    </div>
    
    <script type="text/javascript" src="candy-shop.js"></script>
  </body>
</html>
```

In the head we pull in rivets script. Inside the body we are creating a div with an h1 tag. The h1 tag has `{ data.title }` in it. If you've used templating engines before this should look familiar to you.

#### css

We won't be editing this css at all. I'd just cut and paste it and name the file "candy-shop.css". This will just help the app fit on your screen a bit nicer, but is not in any way necessary for this tutorial.

```css
ul {
  margin-left: 0;
  padding-left: 0;
}

li {
  display: inline-block;
  padding: 0 20px;
  border-left: 1px dotted grey;
}

li:first-child {
  padding-left: 0;
  border-left: none;
}
```

#### js

This is where things will start to get fun. Name this file "candy-shop.js".

```js
var data = {
    title: 'Welcome to the Candy Shop'
    };

rivets.bind(
  document.querySelector('#candy-shop'), // bind to the element with id "candy-shop"
  {
  data: data // add the data object ao we can reference it in our template
});
```

To start, all we'll do is create a data object with a key/value pair for our title. Then we use the `rivets.bind()` method to associate our data with a DOM element. The first argument tells rivets where in the DOM to bind to, in this case we're binding to the element with id "candy-shop". Then we pass in an object. This is the object we'll have access to when templating content inside the bound element. We'll add our data object to it so we can reference it within the #candy-shop div.

### One Way Binding

Now open index.html in your browser. You should see your h1 populated with the "Welcome to the Candy Shop" string.

Now open the console in your browsers developer tools. Type `data.title = 'I have achieved one way binding'`. Notice as soon as you hit enter, the contents of the h1 change to the value of `data.title`. This is one way binding. Any changes to your data object will immediately update the DOM.

### Binders

Rivets comes with an assortment of built in [binders](https://github.com/mikeric/rivets/wiki/Core-Binders). These are html attributes you can add to DOM elements to associate data with them in different ways. In rivets all binders begin with an `rv-` prefix, not at all different then the [html5 "data-" prefix](https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/Using_data_attributes) we know and love.

One of the simplest binders is `rv-text`. It does essentially the same thing as the handlebars-like templating we just did. Update the h1 tag in your html file to use the rv-text binder.

```html
      <h1 rv-text="data.title"></h1>
```

Refresh your browser and see everything is the same as before.

#### Looping Through an Array with the rv-each-* Binder

A common task in templating is generating markup from an array. Rivets uses the `rv-each-*` [binder](https://github.com/mikeric/rivets/wiki/Core-Binders#each-item) for this. This employs an interesting trick where the variable you define in the glob `rv-each-<variable>` becomes your reference to that object within the loop (in the case the object is an array). This is analagous to [for...in](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in) loop logic, where `for variable in object` is analagous to `rv-each-variable="object"`.

Let's define the array of products for our candy shop. We'll add this to the data object in "candy-shop.js". We're going to use all the items named in the 50 cent song ["Candy Shop"](http://rapgenius.com/50-cent-candy-shop-lyrics).

```js
var data = {
    title: 'Welcome to the Candy Shop',
    
    products: [
      {
        title: 'lollipop',
        price: 0.5
      },
      {
        title: 'teakettle',
        price: 14.95
      },
      {
        title: 'magic stick',
        price: 89.12546
      },
      {
        title: 'low rider',
        price: 22450
      },
      {
        title: 'champagne',
        price: 45
      }
    ]};
```

Now we can reference `data.products` inside the #candy-shop div and write an unordered list of the contents with `rv-each-*`.

```html
    <div id="candy-shop">
      <h1 rv-text="data.title"></h1>
      <h3>Products</h3>
      <ul>
        <li rv-each-product="data.products">
          <h3 rv-text="product.title"></h3>
          <var rv-text="product.price"></var>
      </ul>
    </div>
```

After adding the above code to "index.html", refresh your browser. You'll see the product titles and prices listed out. Notice, however that the numbers do not read as properly formatted prices in USD. This is where formatters come in.

### Formatters

[Formatters](https://github.com/mikeric/rivets/wiki/Example-formatters) are essentially functions, which accept arguments and returns a value. Something any developer is well familiar with. In this case we'll be accepting a number and returning a formatted html string. Let's write a formatter called price. You can place this in your js anywhere before your `rivets.bind()` function.

```js
rivets.formatters.price = function(val) {
  
  var spl = String(val).split('.'),
    dollarsArray = spl[0].split(''),
    lastDollar = dollarsArray.length - 1,
    pow,
    i;
  
  if(dollarsArray.length > 3) {
    
    dollarsArray.reverse();
    
    for(i = lastDollar; i > -1; i--) {
      
      if(i % 3 === 0 && i !== 0) {
        
        dollarsArray.splice(i, 0, ',');
      }
    }
    
    spl[0] = dollarsArray.reverse().join('');
  }
  
  if(spl.length > 1) {
    
    spl[1] = spl[1].substr(0, 2);
    
    if(spl[1].length < 2) {
      
      spl[1] += '0';
    }
  }else {
    
    spl[1] = '00';
  }
  
  return '<abbr title="USD">$</abbr>' + spl.join('.');
};
```

In the above code you'll see we accept a numeric value. Do some logic to format the price into a readable USD denomination, and return a string with some html markup.

#### _Disclaimer!_

_The topics of how best to format a price, and the most efficient way to do so are certainly open for debate. The above code is is no way supposed to be a best-practices example of how to do so. It's just there to suit the purpose of explaining formatters within this tutorial._

Now let's apply the price formatter to our price binder. This is done in similar syntax to a Unix pipeline. If you've ever used a pipe in the Mac command line, you already know how to use a formatter. Update your price html to read as follows.

```html
          <var rv-text="product.price | price"></var>
```

In the above code we're taking the value of `product.price` and sending it as an argument to the `price` formatter, then displaying the return value.

Refresh your browser. You should see the html string we return in our price formatter. You'll also notice that there's a small problem. The html is being printed out as plain text instead of being escaped as html.

> <abbr title="USD">$</abbr>0.50

We need to switch to the `rv-html` binder to display our markup properly. This is similar to the difference between [textContent](https://developer.mozilla.org/en-US/docs/Web/API/Node.textContent) and [innerHTML](https://developer.mozilla.org/en-US/docs/Web/API/Element.innerHTML) properties in javascript.

```html
          <var rv-html="product.price | price"></var>
```