---
title: Scoped Bindings in SpecFlow
tags:
  - .NET
  - dev
  - SpecFlow
  - Testing
language: default
date: 2015-08-03 12:12:43
---

One of the things that I found slightly problematic about [SpecFlow](http://www.specflow.org/) is the global scoping of step definitions within bound classes, specifically when there are multiple feature files referring to shared step definitions.

**So why do I feel this way?**

Well, firstly there’s the reduced discoverability of step definitions. Sure you can use the build-in SpecFlow navigation feature to get to the definition, but this seems a little inefficient to me. I guess you could also use [ReSharper](https://www.jetbrains.com/resharper/), but not everyone has this luxury. My other main gripe is that fact that you can’t use (or at least have to be careful using) private fields to maintain state for individual step definitions. The alternative is to use the ScenarioContext, which is a little over complicated for mine:

<pre lang="csharp">
[Given(@"a property price of (.*)")]
public void GivenAPropertyPriceOf(Decimal propertyPrice)
{
    ScenarioContext.Current.Set(propertyPrice, "propertyPrice");
}

[When(@"we press the button")]
public void WhenWePressTheButton()
{
    var propertyPrice = ScenarioContext.Current.Get<Decimal>("propertyPrice")
    // ...
}
</pre>

**The solution – scoped bindings!**

Scoped bindings allow the developer to have a one-to-one mapping between feature files and the step definitions defined by that feature. Sure we lose some reusability of individual step definitions, but in my opinion the pros outweigh the cons. The approach I’ve taken is to have my generated class files scoped to an associated feature as shown below:

<pre lang="csharp">
    [Binding]
    [Scope(Feature = "Name of Feature")]
    public class Test
    {
        private decimal _propertyPrice;
        private decimal _propertyValue;

        [Given(@"a property price of (.*)")]
        public void GivenAPropertyPriceOf(Decimal propertyPrice)
        {
            _propertyPrice = propertyPrice;
        }

        [Given(@"a property value of (.*)")]
        public void GivenAPropertyValueOf(Decimal propertyValue)
        {
            _propertyValue = propertyValue;
        }

        [When(@"we press the button")]
        public void WheneWePressTheButton()
        {
            // ...
        }

        [Then(@"the result is (.*)")]
        public void ThenTheResultIs(Decimal result)
        {
            // ...
        }
    }
</pre>

Now I've got quick and easy discoverability as well as the fact that I can use private fields without concern.