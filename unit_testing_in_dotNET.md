# Unit testing in .NET Core and .NET Standard

Unit tests test individual software components and methods. Unit tests should only test code within the developer’s control. They should not test infrastructure concerns. Infrastructure concerns include databases, file systems, and network resources.

Test Driven Development (TDD) is when a unit test is written before the code it is meant to check. TDD is like creating an outline for a book before we write it. It is meant to help developers write simpler, more readable, and efficient code.

Try not to introduce dependencies on infrastructure when writing unit tests. These make the tests slow and brittle, and should be reserved for integration tests. You can avoid these dependencies in your application by following the Explicit Dependencies Principle and using Dependency Injection. You can also keep your unit tests in a separate project from your integration tests. This ensures your unit test project doesn’t have references to or dependencies on infrastructure packages.


# Unit testing best practices with .NET Core and .NET Standard

## **Why unit test?**
* Less time performing functional tests
* Protection against regression
* Executable documentation
* Less coupled code


## **Characteristics of a good unit test**

* **Fast**. It is not uncommon for mature projects to have thousands of unit tests. Unit tests should take very little time to run. Milliseconds.
* **Isolated**. Unit tests are standalone, can be run in isolation, and have no dependencies on any outside factors such as a file system or database.
* **Repeatable**. Running a unit test should be consistent with its results, that is, it always returns the same result if you do not change anything in between runs.
* **Self-Checking**. The test should be able to automatically detect if it passed or failed without any human interaction.
* **Timely**. A unit test should not take a disproportionately long time to write compared to the code being tested. If you find testing the code taking a large amount of time compared to writing the code, consider a design that is more testable.


## **Terms**

The term mock is unfortunately very misused when talking about testing. The following defines the most common types of fakes when writing unit tests:

* `Fake` - A fake is a generic term which can be used to describe either a stub or a mock object. Whether it is a stub or a mock depends on the context in which it's used. So in other words, a fake can be a stub or a mock.
* `Mock` - A mock object is a fake object in the system that decides whether or not a unit test has passed or failed. A mock starts out as a Fake until it is asserted against.
* `Stub` - A stub is a controllable replacement for an existing dependency (or collaborator) in the system. By using a stub, you can test your code without dealing with the dependency directly. By default, a fake starts out as a stub.

Consider the following code snippet:
```csharp
var mockOrder = new MockOrder();
var purchase = new Purchase(mockOrder);

purchase.ValidateOrders();

Assert.True(purchase.CanBeShipped);
```

This would be an example of stub being referred to as a mock. In this case, it is a stub. You're just passing in the Order as a means to be able to instantiate Purchase (the system under test). The name MockOrder is also very misleading because again, the order is not a mock.

A better approach would be:
```csharp
var stubOrder = new FakeOrder();
var purchase = new Purchase(stubOrder);

purchase.ValidateOrders();

Assert.True(purchase.CanBeShipped);
```

By renaming the class to `FakeOrder`, you've made the class a lot more generic, the class can be used as a mock or a stub. Whichever is better for the test case. In the above example, `FakeOrder` is used as a stub. You're not using the `FakeOrder` in any shape or form during the assert. `FakeOrder` was just passed into the Purchase class to satisfy the requirements of the constructor.

To use it as a Mock, you could do something like this:
```csharp
var mockOrder = new FakeOrder();
var purchase = new Purchase(mockOrder);

purchase.ValidateOrders();

Assert.True(mockOrder.Validated);
```

In this case, you are checking a property on the Fake (asserting against it), so in the above code snippet, the `mockOrder` is a Mock.

The main thing to remember about mocks versus stubs is that mocks are just like stubs, but you assert against the mock object, whereas you do not assert against a stub.


## **Best practices**

### **Naming your tests**
The name of your test should consist of three parts:
* The name of the method being tested.
* The scenario under which it's being tested.
* The expected behavior when the scenario is invoked.

### **Arranging your tests**

Arrange, Act, Assert is a common pattern when unit testing. As the name implies, it consists of three main actions:
* Arrange your objects, creating and setting them up as necessary.
* Act on an object.
* Assert that something is as expected.

### **Write minimally passing tests**
The input to be used in a unit test should be the simplest possible in order to verify the behavior that you are currently testing.

Tests that include more information than required to pass the test have a higher chance of introducing errors into the test and can make the intent of the test less clear. When writing tests you want to focus on the behavior. Setting extra properties on models or using non-zero values when not required, only detracts from what you are trying to prove.

### **Avoid magic strings**

Naming variables in unit tests is as important, if not more important, than naming variables in production code. Unit tests should not contain magic strings.

When writing tests, you should aim to express as much intent as possible. In the case of magic strings, a good approach is to assign these values to constants.

### **Avoid logic in tests**

When writing your unit tests avoid manual string concatenation and logical conditions such as if, while, for, switch, etc.

When you introduce logic into your test suite, the chance of introducing a bug into it increases dramatically. The last place that you want to find a bug is within your test suite. You should have a high level of confidence that your tests work, otherwise, you will not trust them. Tests that you do not trust, do not provide any value. When a test fails, you want to have a sense that something is actually wrong with your code and that it cannot be ignored.

### **Prefer helper methods to setup and teardown**

If you require a similar object or state for your tests, prefer a helper method than leveraging Setup and Teardown attributes if they exist.

In unit testing frameworks, `Setup` is called before each and every unit test within your test suite. While some may see this as a useful tool, it generally ends up leading to bloated and hard to read tests. Each test will generally have different requirements in order to get the test up and running. Unfortunately, `Setup` forces you to use the exact same requirements for each test.

### **Avoid multiple asserts**

When writing your tests, try to only include one Assert per test. Common approaches to using only one assert include:
* Create a separate test for each assert.
* Use parameterized tests.

