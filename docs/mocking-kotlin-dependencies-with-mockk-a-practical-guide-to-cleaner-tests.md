---
layout: default
title: "Mocking Kotlin Dependencies with MockK: A Practical Guide to Cleaner Tests"
description: "Simplify your Kotlin unit testing by mastering MockK for dependency injection and behavior verification. Learn how to stub methods, verify interactions, and use dynamic mocking for robust tests."
permalink: /mocking-kotlin-dependencies-with-mockk-a-practical-guide-to-cleaner-tests/
---

TL;DR: Use MockK to easily create mocks, stub method calls, and verify interactions in your Kotlin unit tests, leading to more maintainable and reliable code.

## The Problem

Testing code that relies on external dependencies can be a major headache.  Imagine you're building a service that interacts with a database or a third-party API. You don't want your unit tests to actually hit the database or API every time they run because it makes them slow, unreliable, and difficult to set up. The answer is to isolate the unit under test by replacing those dependencies with controlled test doubles (mocks).  However, setting up and managing mocks can be cumbersome, especially when dealing with complex interactions.

## The Real-World Scenario

Let's say we're building an e-commerce platform.  We have a `ProductService` that depends on a `InventoryRepository`.  The `ProductService` needs to check if a product is in stock before allowing a customer to purchase it.  Without mocking, a unit test would require setting up a real database, inserting test data, and cleaning up afterward. This adds significant overhead to the testing process.

## The Solution

MockK simplifies this entire process. Let's examine the provided code.  First, consider the `UserService` and `UserRepository` example.

```kotlin
interface UserRepository {
    fun findNickname(id: Long): String
    fun updateLastLogin(id: Long)
}

class UserService(private val repository: UserRepository) {
    fun getWelcomeMessage(id: Long): String {
        val nickname = repository.findNickname(id)
        repository.updateLastLogin(id)
        return "Welcome, $nickname!"
    }
}

class UserServiceTest {
    private val repository = mockk<UserRepository>()
    private val service = UserService(repository)

    @Test
    fun `should return welcome message and update login status`() {
        val userId = 1L
        every { repository.findNickname(userId) } returns "EasyTester"
        every { repository.updateLastLogin(userId) } just Runs

        val result = service.getWelcomeMessage(userId)

        assertThat(result).isEqualTo("Welcome, EasyTester!")

        verify(exactly = 1) { repository.findNickname(userId) }
        verify(exactly = 1) { repository.updateLastLogin(userId) }

        confirmVerified(repository)
    }
}
```

Here's what's happening:

1.  **`mockk<UserRepository>()`**:  We create a mock implementation of `UserRepository`.  This mock behaves exactly as we tell it to within the test.
2.  **`every { repository.findNickname(userId) } returns "EasyTester"`**: This tells MockK that whenever `findNickname` is called with `userId = 1L`, it should return "EasyTester".  This is called *stubbing*.
3.  **`every { repository.updateLastLogin(userId) } just Runs`**:  This tells MockK that whenever `updateLastLogin` is called with `userId = 1L`, it should do nothing. `just Runs` is MockK's way of saying "return void" (or Unit in Kotlin).
4.  **`verify(exactly = 1) { repository.findNickname(userId) }`**: This asserts that `findNickname` was called exactly once with the specified arguments. This is called *verification*.
5.  **`confirmVerified(repository)`**: This asserts that all interactions with the `repository` have been verified. It's a good practice to ensure no unexpected calls happened to our mocks.

Now, consider the `PaymentService` and `PaymentGateway` example demonstrating dynamic stubbing with `answers` and relaxed mocks:

```kotlin
interface PaymentGateway {
    fun process(amount: Int): Boolean
    fun logTransaction(message: String)
}

class PaymentService(private val gateway: PaymentGateway) {
    fun pay(amount: Int): String {
        if (amount <= 0) return "INVALID"

        val success = gateway.process(amount)
        gateway.logTransaction("Processed amount: $amount")

        return if (success) "SUCCESS" else "FAILURE"
    }
}

class PaymentServiceTest {

    @Test
    fun `should process payment using dynamic logic in answers`() {
        val gateway = mockk<PaymentGateway>(relaxed = true)
        val service = PaymentService(gateway)

        every { gateway.process(any()) } answers {
            val amount = it.invocation.args[0] as Int
            amount % 2 == 0
        }

        assertThat(service.pay(10)).isEqualTo("SUCCESS")
        assertThat(service.pay(11)).isEqualTo("FAILURE")

        verify { gateway.process(10) }
        verify { gateway.process(11) }
    }
}
```

Here's what's happening:

1. **`mockk<PaymentGateway>(relaxed = true)`**: We create a *relaxed* mock of `PaymentGateway`. Relaxed mocks automatically return default values for any unstubbed method calls and don't require explicit verification, reducing boilerplate.  In this case, unstubbed calls to `logTransaction` are ignored.
2. **`every { gateway.process(any()) } answers { ... }`**: This allows us to define dynamic behavior based on the input parameters.  In this case, we're returning `true` if the amount is even and `false` if it's odd. The `answers` block gives you full control to execute arbitrary logic.
3.  **`verify { gateway.process(10) }`**: Verifies that process was called with argument 10.

## Key Takeaways

*   **Isolate dependencies:**  Mock external services and databases to create fast and reliable tests that focus on the unit's logic.
*   **Use dynamic stubbing:**  Leverage `answers` to create complex mock behaviors that depend on the input parameters.
*   **Choose the right Mock Type:** Consider relaxed mocks for simplifying tests where some interactions are irrelevant and you want to avoid excessive stubbing.

[Kotlin Unit Testing with JUnit 5](relative-path/Kotlin Unit Testing with JUnit 5.md)

---

**[Get the full Easy Mind guide on Amazon](https://www.amazon.com/dp/B0GDMP4VKD)**

[← Back to Home](https://easy-mind-read.github.io/junit5-mock5-the-art-of-kotlin-testing-pages/)
