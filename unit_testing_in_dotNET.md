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

