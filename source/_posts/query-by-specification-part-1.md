---
title: Query-by-specification part 1 - the problem with repositories
language: default
date: 2016-04-05 19:35:24
tags:
	- Query-by-specification
	- Entity Framework
	- .NET
	- dev
	- design patterns
	- SOLID
---

Instead of writing one massive blog entry, I’ve decided to write this as a series of smaller entries as it’s currently 11:33 PM on a Tuesday night and I don’t really want to be up until 4 AM.

Anyways, I’m going to break it down as follows:
* [Part 1 – the problem with repositories](#part1)
* [Part 2 – specifications to the rescue](#)
* [Part 3 – introducing my own query.by.specification framework](#)

<a name="part1"></a>Many projects that I’ve worked on in the past have a data access layer and use relation databases as the data store. For many years I’ve used object-relation-mappers (ORMs) such as [NHibernate](http://nhibernate.info/) or [Entity Framework](https://msdn.microsoft.com/en-au/data/ef.aspx) to solve the impedance mismatch problem and to allow me to work with a level of persistence ignorance.

My usual approach for the data access layer has been to follow the [repository pattern](http://martinfowler.com/eaaCatalog/repository.html). This pattern is great for simple well understood domains with simple data models, but from my experience it’s not ideal for when the data model is complex and/or queried in many different ways. In addition the repository pattern can violate some core object oriented design rules such as:
* [single-responsibility-principle](https://lostechies.com/seanchambers/2008/03/15/ptom-single-responsibility-principle/) (SRP)
* [open-closed-principle](https://lostechies.com/joeocampo/2008/03/21/ptom-the-open-closed-principle/) (OCP)
* [don’t-repeat-yourself](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) (DRY)

SRP and OCP are part of the [SOLID](https://www.pluralsight.com/courses/principles-oo-design) principles.

Taking everything above into account, it’s still important to remain pragmatic – the repository pattern is a good de-facto approach for most data access layers. Only when there is a high level of uncertainty, change and complexity should we look for alternatives.

Let us explore this a little bit deeper with an example. I’m currently working for 'a bank' and there is the concept of 'loans'. The loans data model isn’t overly complex, but evidently there are many ways of querying for loans. In the last few months alone, I've had to update the repository with several new query operations - a clear violation of OCP. In regards to refactoring for OCP, my usual approach is to follow the 'fool me once shame on you, fool me twice shame on me' proverb. Basically what this means is that I don't initially build software conforming to OCP as it's something that usually incurrs a bit of a cost in terms of time and complexity - in a word it's YAGNI. If when I do need to change a class, the first time is 'shame on you'. If I need to change that same class a second time; 'shame on me' (and time to refactor for OCP). Anyway, back to my scenario...

If we go the vanilla repository pattern route we might have an initial loan repository abstraction as follows (obviously this is based on the initial requirements of the system):

<pre><code class='language-cs'>public interface ILoanRepository
{
    Loan GetLoanById(int id);
}
</code></pre>

Let’s say a new requirement comes in that requires the creation of a new service operation to get all the loans for a particular branch, region, minimum amount in arrears, instalment amount, or some combination these...

<pre><code class='language-cs'>public interface ILoanRepository
{
    Loan GetLoanById(int id);
    IEnumerable<Loan> GetLoansByBranchName(string branchName);
    IEnumerable<Loan> GetLoansByRegion(string region);
    IEnumerable<Loan> GetLoansByMinimumAmountInArrears(decimal minimumAmountInArrears);
    IEnumerable<Loan> GetLoansByInstalmentAmount(decimal instalmentAmount);
    IEnumerable<Loan> GetLoansByBranchNameAndMinimumAmountInArrears(string branchName, decimal minimumAmountInArrears);
    IEnumerable<Loan> GetLoansByBranchNameMinimumAmountInArrearsAndInstalmentAmount(string branchName, decimal minimumAmountInArrears, decimal minimumAmountInArrears);
    //...
}
</code></pre>

The repository, which was quite simple initially, is starting to get out of hand. Getting back to the point about object oriented design principles, we can see the implementation starting to get a bit messy violating DRY, SRP and OCP:
* DRY – we have duplication of partial queries and/or predicates for multiple repository operations.
* SRP – the repository implementation is responsible for many different queries for potentially unrelated use-cases.
* OCP – every time we want to query the data model with a different set of criteria, we have to modify the loan repository implementation. OCP states that objects should be closed for modification, but open for extension.

An alternative solution is to use the [specification pattern](https://lostechies.com/chrismissal/2009/09/11/using-the-specification-pattern-for-querying/) in combination with the repository pattern. As we'll see this approach adheres to DRY, SRP and OCP and subsequently improves the simplicity, extensibility, reusability, and testability of the implementation.

This concludes the first part of this series. Stay tuned for the next part; [specifications to the rescue](#).
