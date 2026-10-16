
<p> example of paragrph element
<a> Anchor Tag Example 

**Attribute such as href goes inside tags

• Alt Attribute
<img src="... " alt="Image example "   >

[ Comments ]

<!--    ... -->

use: Command + /   For a shortcut

<div> </div> Content Division Element ==== Block Element

<span> </span>  ====== inline element 


<hr> a thematic break between paragraph elements - a horizontal line that can be styled across the page.


<br> line break element

<sup> </sup> superscript element = places text above the baseline of normal text. Used in cases 													such as displaying exponents or citations ( wikipedia ).

<sub> </sub> subscript element = places text below the baseline of normal text. Used in cases such as displaying chemical compounds or chemistry formulas


[ HTML Entities ]

Special codes that will result in different characters. ( copyright symbol, diamond, cross, arrow)

Start with ampersand and end in semi colon, with characters in between 


## Semantic Markup

Function exactly like "< div >"element

1.<main> = main content of the page.

2. <header> = header of the page

3. <nav> = navigation elements

4. <footer> = footer element

5. <section> = section element

6. <article> = self contained content that is reusable 

7. <aside> = content that is indirectly related to the main body of the page.

8. < time> = notes the time. also includes the datetime 
	attribute.
9. <figure> = visual image with content that stands alone. includes things such as diagrams or some visual you want to draw attention to. 

10. <abbr>

11. <data>

Using semantics allows for easier interpretation and making the content more accessible to crawlers and users. Semantic Elements are more specific than the div element. 


## Table 

<table> create table

<tr> table row
<td> table data "cell"
<th> header element cell in the header
<theader> header in table
<tbody> body of the table
<tfooter>footer of the table

<th rowspan = "2"> header cell takes up 2 rows with rowspan attribute
<th colspan = "2"> header cell takes up 2 columns with colspan attribute


## Forms

<form>
- represents a **document** section containing interactive controls for submitting information
- the **action** attribute specifies where the form data should be sent
- the **method** attribute specifies which HTTP method should be used 

- action attribute acts like an address to send forms to



<input>
- input element is used to create a variety of different form controls
- 20+ possible types of inputs ranging from date pickers to checkboxes.
- the *type* attribute is where the magic happens changing type dramatically alters the input's behavior and appearance. 

		<input type=""> type attribute defines the type of input that will be entered

		<input placeholder=""> placeholder attribute defines text that will take place of input (before input) in the input field

## Buttons

<button> Button Text </button>

Buttons inside the <form> element will be automaticaly assigned to submit the form

## Name Atttribute

name="" attribute assigns a name to the element

ie: for <input> elements to label the data with a name when the data is sent to the server


- If there is no input element in the form submission and you hit enter the form will STILL be submitted

## Input Types 

*Checkbox Input Type*

<input type="checkbox"> use the checkbox input type to create a checkbox for the user
			use the checked attribute to auto-check the checkbox


*Radio Input Types*


<input type="radio"> use a label to title the radio button. 

*value = " "*the value attribute defines what will be sent through when the option is selected

<select name= id= > creates a drop down menu with <option value=""> elements

the selected attribute for option will pre-select an option in the dropdown

**Range Input Types**

<input type="range" > creates a slider input that 
	use min and max attributes to select boundaries for the range
		use the step attribute to set what the selector incriment as one interacts with the slider
		use the value attribute to set the initial value that the slider will start on

<input type ="number"> creates an input box that takes numerical values as input
		use the min and max attributes to set the boundaries for the number input
		can also use step attribute to set incriment level

<textarea>  element creats a text box 
	- use the rows attribute to select the amount of rows in the text area
	- use the cols attribute to select the amount of columns in the text area

## Form validations

Required Attribute
	- Place inside of input element
	- will also reinforce other attribute such as minlength and maxlength and make them part of the required paramters for the input

**Pattern Attributes**
- using the 
- Use the minlength and maxlength attributes to handle the length of the input text

- With the required attribute and different input types the html is merely looking for a pattern (if email type looks for text@text ) not verifying the input exactly

## Href

Href attribute determines the path of the file that is being linked