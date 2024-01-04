---
layout: post
title: "Endless Methods and Implicit Hash Values"
date: 2023-10-16
categories: Ruby Tips
---

_This post was first published on the [Brewer Digital Engineering Blog](https://brewerdigital.com/engineering/say-no-to-repetitive-code-with-endless-methods-and-implicit-hash-values-in-ruby/)_


<h1>Fun New Feature in Ruby 3</h1>

Ruby 3.0.0 was released on Christmas Day, 2020, with the 3.3 update coming this Christmas. The major release came packed with lots of new features and improvements, including the exciting YJIT project. But some of my favorite features have been the quality of life improvements, the little pieces of syntax that make Ruby a joy to write. Two of these features are endless methods and implicit hash values.

<h2>Endless Methods</h2>

Endless methods are very literally named: it’s a one-line method definition that doesn’t need to terminate with end.

Great use cases for them include getter-type methods:

```
def tip_rate = 0.2
```

simple calculations:

```
def calculate_tip(amount) = amount * tip_rate
```

or ActiveRecord queries:

```
def receipts = Document.where(type: 'receipt')
```

These endless methods have also been helpful in writing, and maintaining, good code, as they encourage the single-responsibility principle.

The one-line syntax encourages the method to be simple and concise. It is hard to tackle multiple responsibilities in a single line. Then as the application grows, the syntax discourages scope creep, since adding lines to the method requires rewriting the definition. Reformatting the syntax is easy enough to do when the use-case demands, but it provides just enough friction to reduce the sort of method expansion that can happen as codebases mature.

<h2>Omitting hash literal values</h2>

Implicit hash literal values is a feature that has been in JavaScript for some time, but has now come to Ruby and fits right in with the [syntactical sugar](https://phillipstrom.com/ruby/rspec/2022/04/06/rspec-magic.html) the language has already embraced.

When dealing with key/value pairs, previously we had to always write out the key and the value.

```
tip_rate = 0.2
tax_rate = 0.081
additional_fees = { tip_rate: tip_rate, tax_rate: tax_rate }
```

Now if the key name matches the variable name, Ruby will implicitly understand what value the key is referencing, so the example above can be written to look like this:

```
tip_rate = 0.2
tax_rate = 0.081
additional_fees = { tip_rate: , tax_rate: }
```

This syntax has also been beneficial in writing readable code. Using this shortcut encourages consistent naming conventions, so the value you are passing is clearly matched with the key to which it belongs.

<h2>Combining Endless Methods with Hash Literals</h2>
These two features end up pairing together nicely as the omitted value works with method names. So you can follow Object-Oriented principles by encapsulating your data in logic, and then pass these method calls into hashes seamlessly as implicit values.

```
def tip_rate = 0.2
def tax_rate = 0.081
def serialize_rates = { tip_rate: , tax_rate: }.to_json
```
