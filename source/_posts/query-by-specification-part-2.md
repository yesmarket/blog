---
title: Query-by-specification part 2 - specifications to the rescue
language: default
date: 2016-05-09 11:10:42
tags:
	- Query-by-specification
	- Entity Framework
	- .NET
	- dev
	- design patterns
	- SOLID
---

This is part 2 of the 'query-by-specification' series:
* [Part 1 – the problem with repositories](/2016/04/query-by-specification-part-1/)
* [Part 2 – specifications to the rescue](#part2)
* [Part 3 – introducing my own query.by.specification framework](#)

<a name="part2"></a>In the previous part of this series, we looked at some of the limitations of the repository pattern and why this might not be the best approach in certain situations. In this part we'll take a look at the specification pattern, specifically in the context of data access layers.

I've often encountered scenarios where I need to implement a large number of different queries against a single entity. This scenario might happen because of a single user-story such as some kind of search feature where there is a large number of searchable criteria. Alternatively, it might arise following multiple independent user-stories that each require access to some common entity (but with different filter criteria). In these situations, how do we ensure our data access code doesn't degrade into 1000+ LOC repositories that violate SOLID and DRY? The answer to this question is 'specifications'.

The specification pattern requires a slight change in mindset from the traditional repository pattern approach. Instead of thinking in terms of whole queries we need to think in terms of common sets of filter criteria. These common sets of filter criteria are basically the specifications. 

Going back to our 'loan' example from the previous part of this blog series, we might be able to define specifications as follows:
- BranchNameSpecification
- RegionSpecification
- ArrearsAmountSpecification
- InstalmentAmountSpecification

Our repository, instead of having methods for every combination of filter criteria, now has a single function to get loans by a specification as shown below:

    public interface ILoanRepository
    {
    	IEnumerable<Loan> FindBy(ISpecification<Loan> specification);
    }

Note: <code lang="sh" linenumbers="off">ISpecification<Loan></code> can be a composite specification ('AndSpecification', 'OrSpecification' etc.) such that multiple filter criteria can be chained together. Depending on the implementation you might define some kind of [internal DSL](http://martinfowler.com/books/dsl.html) to make things a bit easier to use. 

The following example shows how you might get loans from the 'East' region that are in arears by more than $100 using a crude [fluent interface](http://martinfowler.com/bliki/FluentInterface.html):

    var loans = _loanRepository.FindBy(new RegionSpecification("East").And(new ArrearsAmountSpecification(Operator.GreaterThan, 100M)));

The <code lang="sh" linenumbers="off">ISpecification<T></code> interface might vary depending on how you want to implementat things. As we'll see in the next part of this blog series, I like to define the <code lang="sh" linenumbers="off">ISpecification<T></code> contract as follows:

    public interface ISpecification<T>
    {
    	Expression<Func<T, bool>> GetPredicate();
    }

My aim was to keep things as simple as possible. Basically the <code lang="sh" linenumbers="off">GetPredicate()</code> implementation just needs to define a strongly typed lambda expression to determine if an object matches some arbitrary criteria. Simples

<img style="float: left;" src="/images/meerkat.png">

<div style=" clear: both;">
The following example shows how a hypothetical region specification might look:
</div>

    public class RegionNameSpecification : BaseSpecification<Loan>
    {
    	public RegionNameSpecification(string name)
    	{
    		Name = name;
		}

		public string Name { get; set; }

		public override Expression<Func<Loan, bool>> GetPredicate()
		{
			return loan => loan.RegionName == Name;
		}
	}

To contrast this with the repository approach we can see that this approach now adheres to DRY, SRP and OCP.
* DRY – we no longer need to duplicate partial queries and/or predicates.
* SRP - the repository implementation has a single responsibility, which is to find entities that match the defined specification.
* OCP – we should never have to modify the repository. If we need to query by some undefined filter criteria, we simply add a new specification.

Before we wrap up this part of the series, it would be biased of me to not mention any of the drawbacks to the specification pattern - this is not a panacea to be used for all data access layers. Careful consideration should be given to whether or not it is appropriate for your scenario and that the pros outweigh the cons. Some of the 'not-so-good' things about the specification pattern are as follows:
* While the repository pattern is quite natural and very easy to implement, the specification patttern incurs a bit of a overhead both terms of a) ease of understanding and b) the need to implement some boilerplate. In the case of the latter; sure there are many existing query-by-specification frameworks, however, it's still something extra that you and your team would have to master.
* Some critiques of the specification pattern point to the fact that the repository 'find-by' function is a little bit too open ended and doesn't define an explicit contract for use from the business layer. While I agree with this in principle, sometimes it comes down to the lesser of two evils.