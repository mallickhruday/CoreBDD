# SimpleBDD

BDD framework for xUnit.net

## Getting started with SimpleBDD

Following the usual calculator example, we can start with the following model to test
 
   
  ``` csharp
public class Calculator
    {
        public int Add(int x, int y) => x + y;
        public int Subtract(int x, int y) => x - y;
    }
```

We can define a *Feature* to collate a suite of scenarios by deriving from the *Specification* base class and decorating with the *Feature* attribute

  ``` csharp
    [Feature("Calculator", 
    @"In order to avoid silly mistakes
    As a math idiot
    I want to be told the sum of two numbers")]
    public class CalculatorFeature : Specification
    {
         public CalculatorFeature(GivenWhenThenFixture context):base(context) 
         {
            
         }
    }
```

To create a scenario for this feature, simply inherit from the new Feature class, decorate with a Scenario attribute and provide Given, When, Then methods that will execute in order

  ``` csharp
    [Scenario("Add two numbers")]
    public class AddTwoNumbers : CalculatorFeature
    {
        readonly Calculator calc;

        public AddTwoNumbers(GivenWhenThenFixture state) 
            : base(state) => calc = new Calculator();

        [Given("I have entered {0} into the calculator", 1)]
        public void Given(int first) => Context.Given.First = first;

        [And("I have also entered {0} into the calculator", 2)]
        public void And(int second) => Context.Given.Second = second;

        [When("I press add")]
        public void When() => Context.When = calc.Add(Context.Given.First, Context.Given.Second);

        [Then("the result should be {0}", 3)]
        public void Then(int result) => Context.Result.ShouldBe(result);
    }
```

The above shows a simple, terse implementation using expression bodied members for the Given/When/Then implementation. A more verbose example may look like

  ``` csharp
    [Scenario("Subtract two numbers")]
    public class SubtractTwoNumbers : CalculatorFeature
    {
        readonly Calculator calc;

        public SubtractTwoNumbers(GivenWhenThenFixture state) : base(state)
        {
             calc = new Calculator();
        }

        [Given("I have entered {0} into the calculator", 5)]
        public void Given(int first)
        {
            Context.Given.First = first;
        }

        [And("I have also entered {0} into the calculator", 2)]
        public void And(int second)
        {
            Context.Given.Second = second;
        }

        [When("I press minus")]
        public void When()
        {
            Context.When = calc.Subtract(Context.Given.First, Context.Given.Second);
        }

        [Then("the result should be {0}", 3)]
        public void Then(int result)
        {
           Context.Result.ShouldBe(result);
        }

    }
```

You can also define senarios in a single method using delgates for each of the steps and allowing for multiple scenarios to be defined within the same class

  ``` csharp
    public class AdvancedCalculator : CalculatorFeature
    {
        Calculator calculator;

        [Spec("Multiply two numbers")]
        public void MultiplyTwoNumbers()
        {
            Given("I have a calculator",           () => calculator = new Calculator());
            When("I key in 10",                    () => calculator.Key(10));
            When("I key in 5 and press multiply",  () => calculator.Multiply(5));
            Then("It sets the Total to 50",        () => calculator.Total.ShouldBe(50));
            Then("It sets the equation to 10 x 5", () => calculator.Equation.ShouldBe("10 x 5"));
        }

        [Spec("Divide two numbers")]
        public void DivideTwoNumbers()
        {
            Given("I have a calculator",       () => calculator = new Calculator());
            When("I key in 42",                () => calculator.Key(42));
            Then("It sets the Total to 42",    () => calculator.Total.ShouldBe(42));
            Then("It sets the equation to 42", () => calculator.Equation.ShouldBe("42"));
        }
    }   
```

You can generate Gherkin specs from your tests using the *SimpleBDD.SpecGeneration* extension library, either by calling from an application or command line tool and passing in the path to the assembly containing tests, or by hooking up your test project to generate the specs after the test run. 

To do the latter, create a Fixture class within your test project, reference the *SimpleBDD.SpecGeneration* library and call *GenerateSpecs.OutputFeatureSpecs* within the Dispose method, passing in the Assembly (or path to the Assembly) and the output folder for the generated specs.

  ``` csharp
    [CollectionDefinition("SimpleBDD")]
    public class Collection : ICollectionFixture<GenerateSpecsFixture> { }

    public class GenerateSpecsFixture : IDisposable
    {
        public void Dispose()
        {
            GenerateSpecs.OutputFeatureSpecs(this.GetType().Assembly.Location, @"..\..\..\Specs\");
        }
    }
```

When the tests complete running, a *feature.spec* file is generated under the Specs folder of the xUnit test project. It generates Gherkin specs for the feature and related scenarios. Example *CalculatorFeature.spec* :

  ``` gherkin
    Feature: Calculator
	In order to avoid silly mistakes
    As a math idiot
    I want to be told the sum of two numbers

    Scenario: Add two numbers
    			Given I have entered 1 into the calculator
    			And I have also entered 2 into the calculator
    			When I press add
    			Then the result should be 3

    Scenario: Subtract two numbers
    			Given I have entered 5 into the calculator
    			And I have also entered 2 into the calculator
    			When I press minus
    			Then the result should be 3

```