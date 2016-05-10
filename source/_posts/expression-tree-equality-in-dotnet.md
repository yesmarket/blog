---
title: Expression tree equality in .NET
language: default
date: 2016-05-10 23:47:43
tags:
	- Expression trees
	- .NET
	- dev
---

Today I was refactoring a codebase that I've been working on and I ran into a bit of an interesting problem involving expression trees.

The codebase in question isn't really all that important in terms of this discussion, but to quickly describe my problem; I needed to filter a list of [System.Linq.Expressions.Expression](https://www.google.com.au/webhp?sourceid=chrome-instant&ion=1&espv=2&ie=UTF-8#q=System.Linq.Expressions.Expression) by distinct.

This seemed like a pretty simple thing to do, that is, until I found out that Equals for expressions is implemented as reference equality. This means the following would fail:

    Expression<Func<Order, object>> x = order => order.Customer.Address;
    Expression<Func<Order, object>> y = order => order.Customer.Address;
    Assert.True(x.Equals(y)); // will fail

In my particular scenario, reference equality wouldn't work as I had a list of seperately provided expression trees. I needed a way to determine if these expression trees were essentially the same i.e. by value.

My first thought was to just do a <code lang="cs" linenumbers="off">ToString()</code> on the Expression and to tell you the truth this did kinda work, but it felt dirty... there had to be a better way. What I wanted was a custom expression tree equality comparer - an `IEqualityComparer<Expression>` implementation if you will, preferably on the public nuget.org feed so that I could quickly start using this from within my project. Oh, I should be so lucky.

What I did manage to find was some code from a project called [LINQ to db40](https://sourcecodebrowser.com/db4o/7.4.121.14026plus-pdfsg/class_db4objects_1_1_db4o_1_1_linq_1_1_expressions_1_1_expression_equality_comparer.html) as well as a [blog series](http://badecho.com/2012/02/expression-equality-comparer-part-i/) by a chappy named Matt Weber. Both of these helped give me a really good starting point, but... unfortunately, both of these were for older versions of the .NET framework. If I wanted to get things working, I was going to have to do a bit of refactoring.

For the custom expression tree comparison, I created an <code lang="cs" linenumbers="off">IEqualityComparer<Expression></code> implementation called <code lang="cs" linenumbers="off">ExpressionEqualityComparer</code> as follows:

    public class ExpressionEqualityComparer : IEqualityComparer<Expression>
    {
        public bool Equals(Expression x, Expression y)
        {
            return new ExpressionValueComparer().Compare(x, y);
        }

        public int GetHashCode(Expression obj)
        {
            return new ExpressionHashCodeResolver().GetHashCodeFor(obj);
        }
    }

This is the main class that you would interact with for comparing expression trees. The following example demonstrates a simple use of this class:

    Expression<Func<Order, bool>> x = order => order.Customer.Name == "Joe Bloggs";
    Expression<Func<Order, bool>> y = order => order.Customer.Name == "Joe Bloggs";
    Assert.True(new ExpressionEqualityComparer().Equals(x, y)); // will pass

Most of the gory implementation details are abstracted away in <code lang="cs" linenumbers="off">[ExpressionValueComparer](https://github.com/yesmarket/yesmarket.Linq.Expressions/blob/master/yesmarket.Linq.Expressions/Support/ExpressionValueComparer.cs)</code> and <code lang="cs" linenumbers="off">[ExpressionHashCodeResolver](https://github.com/yesmarket/yesmarket.Linq.Expressions/blob/master/yesmarket.Linq.Expressions/Support/ExpressionHashCodeResolver.cs)</code>. Both of these are subclasses of [ExpressionVisitor](https://msdn.microsoft.com/en-us/library/system.linq.expressions.expressionvisitor.aspx), which can be used to traverse expression trees. During the traversal, specific properties of individual nodes are compared for equality. If any of the relevant properties are not equal, the comparison fails.

Pretty cool, huh!

Anyway one other thing about those resources I mentioned earlier; they were great and all, but I had to do code scraping to pull things together and ultimately they didn't compile under .NET 4.5+, had bits missing e.g. extension methods etc. 

Wouldn't it be great if there was a public git repo and a public nuget package published to the nuget.org feed? [Yes](https://github.com/yesmarket/yesmarket.Linq.Expressions) and [Yes](https://www.nuget.org/packages/yesmarket.Linq.Expressions/).

<div style="float:left;">
	![](/images/oh-yeah.jpg)
</div>