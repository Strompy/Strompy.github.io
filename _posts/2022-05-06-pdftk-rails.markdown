---
layout: post
author: Phillip
title: "Working with PDF Forms in Rails"
date: 2022-05-06 14:31:27 -0600
categories: Ruby Rails pdftk
---

<h1>Using PDFTK and the pdf-forms Gem</h1>

I recently needed a Ruby on Rails backend to fill out PDF Forms, aka those PDFs with editable fields so you can insert text fields, checkboxes, signatures, etc in the file. There weren't too many guides out there about this particular topic, so I wrote down some of what I learned.

The most popular gem (in terms of downloads on RubyGems) for working with these files is pdf-forms. So that is what we will be using in this guide. It offers some basic functionality to read the fields and to fill out the form. Here is a brief overview of how to work with PDF Forms in a Ruby On Rails application.

<h3>SETUP</h3>
First we need a few tools, the aforementioned gem pdf-forms and the command line tool PDFtk.

I am working on MacOS 12.3.1 Monterey and am assuming you have a basic understand of working from the command line as well as Ruby, Ruby on Rails, and Bundler. There are lots of great sources out there for learning those tools (such as [gorails](https://gorails.com/))



<h4>Pdf-forms</h4>


First step is to get the [pdf-forms gem](https://github.com/jkraemer/pdf-forms) in your project.

If you have a gemfile add the following line
```
gem 'pdf-forms'`
```
then install the gem

```
bundle install
```

<br>
Otherwise install the gem via the comand line
```
gem install pdf-forms
```
and include the gem in your ruby file by adding
```
require pdf-forms
```

<h4>PDFtk</h4>
Now since the gem is a wrapper for the PDFtk CLI we will need to install that as well. [PDFtk](https://www.pdflabs.com/tools/pdftk-the-pdf-toolkit/) is a tool from PDFLabs that allows you to interact with PDFs from the command line. The link on the page takes you to an older version of the tool which didn’t work for me on macOS Catalina 10.15. The answer on this [StackOverflow thread](https://stackoverflow.com/questions/60859527/how-to-solve-pdftk-bad-cpu-type-in-executable-on-mac) provides a link to the updated Mac package, which you can download [here](https://www.pdflabs.com/tools/pdftk-the-pdf-toolkit/pdftk_server-2.02-mac_osx-10.11-setup.pkg).


After installing PDFtk you can explore it in the command line. I mostly tried out the `dump_data_fields` command which you run with the prompt:
```
pdftk your_document.pdf dump_data_fields
```
Replace `your_document.pdf` with the file path for the PDF you want to explore. `dump_data_fields` will print out the metadata for the fields on the PDF. For example:
```
---
FieldType: Text
FieldName: Buyer
FieldNameAlt: Buyer
FieldFlags: 5768935
FieldJustification: Left
---
FieldType: Button
FieldName: Joint Buyer
FieldNameAlt: Property
FieldFlags: 0
FieldValue: Off
FieldJustification: Left
FieldStateOption: Off
FieldStateOption: On
---
```


With that set up you and working are ready to go!

<h3>USING THE GEM</h3>
When instantiating an instance of the PdfForms class, it takes in one argument which is the file path to PDFtk in your environment. I did not do anything special when installing so my path was the default: `'/usr/local/bin/pdftk'`

Which means I create a new PdfForms instance like so:
```
form_reader = PdfForms.new('/usr/local/bin/pdftk')
```

<h3>READING PDF FORMS</h3>
With our new PdfForms instance created, the next step is to to read the fields on the pdf form.

There are two helpful methods given to us from the gem:

<h4>get_fields</h4>

`get_fields` returns an array of Field objects that each hold the information about an individual field on the PDF.
`get_fields` takes a single argument, the whole file path to the pdf form you want to read.

```
form_reader.get_fields('your_document.pdf')
```

Returns:
```ruby
[
#<PdfForms::Field:0x00007f7bfde14c10
  @flags="5768935",
  @justification="Left",
  @name="Buyer",
  @name_alt="Buyer",
  @type="Text"
>,
 #<PdfForms::Field:0x00007f7bbfc3cdb8
   @flags="0",
   @justification="Left",
   @name="Joint Buyer",
   @name_alt="Property",
   @options=["Off", "On"],
   @type="Button",
   @value="Off"
   >,
   ...
]
```

You can see this same info matches what `pdftk dump_data_fields` returned, but now conveniently turned into Ruby objects for us to interact with.

<h4>get_field_names</h4>

`get_field_names` does what it sounds like, it returns an array of the name attributes from all those field objects.

```
form_reader.get_field_names(‘your_document.pdf)
```
Returns:
```ruby
[
“Buyer”,
“Joint Buyer”
]
```

<h3>WRITING TO PDF FORMS</h3>
Now the fun part is writing your custom data onto the pdf. Use the method `fill_form` to do this.
It takes several arguments, including an optional one.

First is the full file path to the pdf form you are using.

Second it takes a name of to be used as for the filled out PDF form (which will be a new file in your project’s root directory, the original pdf form will be unchanged).

Third is a hash of the data you want filled in.

Lastly is an optional arguments hash. Here is an example without the optional arguments:

```ruby
file_path = "your_document.pdf"
new_file_name = "completed_form.pdf"
form_data = {
                 "Buyer" =>"Orson Welles",
                 "Joint Tenants" =>"On",
}
form_reader.fill_form(file_path, new_file_name, form_data)
```


To build the hash with the field values (called  `form_data` above), the key will be the name of the field (exactly as it appears in the pdf metadata) and the value will be a string of whatever data you want added to the form. Neither pdftk nor the gem return an error for mismatched field names, it will silently fail by not filling out the misnamed field.

> Hash keys need to be exactly the same as the FieldName on the PDF

The tricky part is working with other field types. Checkboxes have two `options` defined on the PDF form. One of them will make the box checked and the other will make it unchecked. A common one is ‘Yes’ and ‘Off’. Why? I don’t know. But this is where reading the field data will be necessary either with `form_reader.get_fields` or from the command line with `pdftk dump_data_fields`. Each checkbox should have two `FieldTypeOptions` defined on it. That is your source of truth for that specific field. When using the gem you can see these combined into an array and stored as `Field.options`.

.[More info about how to check checkboxes](https://github.com/jkraemer/pdf-forms/issues/22)

> FieldTypeOptions is your source of truth for checkbox values

On my test PDF form, I had a field with unchecked defined as `'Off'` and checked defined as `'Checkbox:plus:option:plus:2'` whatever that means…So you will have to check your own PDF’s fields to confirm what values you need.

One final quick note, when working with Signature field types I have not found a way to write a value with the gem or PDFtk. It seems that there are some additional layers that Adobe has attached to signature fields for security purposes. If you find a way to write to Signature fields, let us know!


There is a quick overview of how to work with PDF Forms in Rails using the pdf-forms gem. Hopefully this is helpful!
