---
layout: post
title: Go interfaces
date: 2019-12-29 15:09:21.000000000 +00:00
type: post
---
My traditional understanding of interfaces was for Polymorphism, i.e where a object of a certain type (as determined by it’s class inheritance) could also be a different type as determined by an interface so long as it implemented the methods in that interface. The concept is similar in golang albeit more lightweight. For example, let’s say I declare two struct types called Circle and Square as follows:

{%highlight go%}
package main

type Square struct { 
  Length float64 
  Width float64 
} 

type Circle struct { 
  Radius float64 
}
{%endhighlight%}

I need a function for each of these types to calculate their area. Because calculating the area of a square is different from calculating the area of a circle these functions are specific to each type, so we’ll use receiver functions to attach an Area to each type.

{%highlight go%}
func (c *Circle) Area() float64 { 
  return math.Pi * c.Radius * c.Radius 
}


func (s *Square) Area() float64 { 
  return s.Length * s.Width 
} 
{%endhighlight%}

<p><!-- /wp:preformatted --></p>
<p><!-- wp:paragraph --></p>
<p>I can call each function to calculate the area as follows:</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:preformatted --></p>
<pre class="wp-block-preformatted">func main() { 
&nbsp;&nbsp;s := &amp;Square{17, 16} 
&nbsp;&nbsp;c := &amp;Circle{15} 

&nbsp;&nbsp;fmt.Printf("The area of s is %f\n", s.Area()) 
&nbsp;&nbsp;fmt.Printf("The area of c is %f\n", c.Area()) 
} </pre>
<p><!-- /wp:preformatted --></p>
<p><!-- wp:paragraph --></p>
<p>So how do interfaces come into play?. Well, what if I want to add the two areas together?. The ideal solution for this would be to send an array of all shapes, iterate through them to call the Area method on each shape, add the results and return the value. The problem is, we cannot have an array (slice) containing different types. What we can do however is define an interface. This interface would be a list of methods that are common to the shapes, i.e both our Circle and Square types have a Area method, so we can define a new interface which declares this method as follows:</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:preformatted --></p>
<pre class="wp-block-preformatted">type Shape interface { 
&nbsp;&nbsp;Area() float64 
} </pre>
<p><!-- /wp:preformatted --></p>
<p><!-- wp:paragraph --></p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:paragraph --></p>
<p>With this we now have a new type called Shape. Any existing type that implements the Area() method will also be of type Shape (plus it's existing type - i.e. just like traditional Polymorphism, i.e. my existing types of Circle and Square are now also of type Shape.  Now we can build an array (slice) of Shapes and iterate through them to call the Area() function and add there areas:</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:preformatted --></p>
<pre class="wp-block-preformatted">func sumAreas(shapes []Shape) float64 { 
&nbsp;&nbsp;total := 0.0 

&nbsp;&nbsp;for _, shape := range shapes { 
&nbsp;&nbsp;&nbsp;&nbsp;total += shape.Area() 
&nbsp;&nbsp;} 

&nbsp;&nbsp;return total 
} </pre>
<p><!-- /wp:preformatted --></p>
<p><!-- wp:paragraph --></p>
<p>Now let’s update the main function as follows:</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:preformatted --></p>
<pre class="wp-block-preformatted">func main() { 
&nbsp;&nbsp;s := &amp;Square{17, 16} 
&nbsp;&nbsp;c := &amp;Circle{15} 

&nbsp;&nbsp;fmt.Printf("The area of s is %f\n", s.Area()) 
&nbsp;&nbsp;fmt.Printf("The area of c is %f\n", c.Area()) 

&nbsp; shapes := []Shape{s,c} 
&nbsp; sumOfAreas :=&nbsp;sumAreas(shapes) 
&nbsp;&nbsp;fmt.Printf(“The sum of areas is: %f\n”,&nbsp;sumOfAreas) 
} </pre>
<p><!-- /wp:preformatted --></p>
<p><!-- wp:paragraph --></p>
<p>Here's the full code:</p>
<p><!-- /wp:paragraph --></p>
<p><!-- wp:preformatted --></p>
<pre class="wp-block-preformatted">package main

import (
  "fmt"
  "math"
)

type Square struct {
  Length float64
  Width  float64
}

type Circle struct {
  Radius float64
}

func (c *Circle) Area() float64 {
  return math.Pi * c.Radius * c.Radius
}

func (s *Square) Area() float64 {
  return s.Length * s.Width
}

type Shape interface {
  Area() float64
}

func sumAreas(shapes []Shape) float64 {
  total := 0.0
  for _, shape := range shapes {
    total += shape.Area()
  }
  return total
}

func main() {
  s := &amp;Square{17, 16}
  c := &amp;Circle{15}
  fmt.Printf("The area of s is %f\n", s.Area())
  fmt.Printf("The area of c is %f\n", c.Area())
  shapes := []Shape{s, c}
  sumOfAreas := sumAreas(shapes)
  fmt.Printf("The sum of areas is: %f\n", sumOfAreas)
}</pre>
<p><!-- /wp:preformatted --></p>
<p><!-- wp:paragraph --></p>
<p><!-- /wp:paragraph --></p>
