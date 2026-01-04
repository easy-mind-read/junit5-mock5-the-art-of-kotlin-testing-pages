---
title: "Conquer Kotlin Coroutines and Static Methods: A Mocking Masterclass"
description: "Tired of wrestling with asynchronous code and static methods in your Kotlin tests? This guide provides practical solutions for mocking coroutines, singleton objects, and extension functions using MockK."
---

TL;DR: Learn how to effectively mock coroutines, singleton objects, and extension functions in Kotlin using MockK to write robust and reliable tests.

## The Problem

Testing asynchronous code and static methods/objects can be a major headache. When dealing with coroutines, traditional testing approaches often fail due to the non-blocking nature of asynchronous operations. Similarly, static methods and singleton objects introduce tight coupling, making it difficult to isolate and test individual components. This often leads to brittle tests that are hard to maintain and don't accurately reflect the behavior of your code.

## The Real-World Scenario

Imagine you're building a data processing pipeline. Your `DataRepository` fetches raw data from a `RemoteDataSource` using a suspending function (coroutines) and then transforms it. You also have a `SecretService` that accesses an API key from a singleton `GlobalConfig` and masks it using an extension function before processing. Now, how do you test these components *without* actually hitting the network, or exposing your real API key?

## The Solution

Let's break down the code and see how MockK comes to the rescue.

**Example 1: Coroutines and Suspending Functions**

```kotlin
interface RemoteDataSource {
    suspend fun fetchData(id: String): String
}

class DataRepository(private val dataSource: RemoteDataSource) {
    suspend fun getProcessedData(id: String): String {
        delay(1000) // Simulating a network delay
        val rawData = dataSource.fetchData(id)
        return rawData.uppercase()
    }
}

class CoroutineTest {
    private val dataSource = mockk<RemoteDataSource>()
    private val repository = DataRepository(dataSource)

    @Test
    fun `should process async data using coEvery and runTest`() = runTest {
        // Given
        val id = "123"
        coEvery { dataSource.fetchData(id) } returns "kotlin-art"

        // When
        val result = repository.getProcessedData(id)

        // Then
        assertThat(result).isEqualTo("KOTLIN-ART")
        coVerify(exactly = 1) { dataSource.fetchData(id) }
    }
}
```

Here's what's happening:

1.  We define a `RemoteDataSource` interface with a suspending function `fetchData`.
2.  The `DataRepository` uses this interface to fetch data and process it (simulating network delay with `delay`).
3.  In the test, we use `mockk<RemoteDataSource>()` to create a mock implementation of the data source.
4.  `coEvery { dataSource.fetchData(id) } returns "kotlin-art"` sets up a coroutine expectation: when `fetchData` is called with "123", return "kotlin-art".
5.  `runTest` ensures that the test waits for the coroutine to complete.
6.  `coVerify` confirms that `fetchData` was called exactly once.

**Example 2: Mocking Object and Extension Functions**

```kotlin
object GlobalConfig {
    fun getApiKey(): String = "REAL-API-KEY"
}

fun String.toMasked(): String = "****"

class SecretService {
    fun processSecret(): String {
        val key = GlobalConfig.getApiKey()
        return key.toMasked()
    }
}

class StaticMockTest {

    @Test
    fun `should mock singleton object and extension function`() {
        val service = SecretService()

        // Mocking a Singleton object
        mockkObject(GlobalConfig)
        every { GlobalConfig.getApiKey() } returns "MOCKED-KEY"

        // Mocking an extension function (must target the class/file)
        mockkStatic("com.easymind.testing.junit5.chapter4.Chapter4_codeKt")
        every { any<String>().toMasked() } returns "MASKED"

        // When
        val result = service.processSecret()

        // Then
        assertThat(result).isEqualTo("MASKED")

        // Cleanup: Mandatory when using static/object mocks to prevent state leaking
        unmockkAll()
    }
}
```

Key points:

1.  `mockkObject(GlobalConfig)` mocks the singleton object `GlobalConfig`.
2.  `every { GlobalConfig.getApiKey() } returns "MOCKED-KEY"` defines the mock behavior for the `getApiKey()` function.
3.  `mockkStatic("com.easymind.testing.junit5.chapter4.Chapter4_codeKt")` mocks the Kotlin file containing the extension function.  Note that you must use the fully qualified name of the file in which the extension function is defined.
4.  `every { any<String>().toMasked() } returns "MASKED"` defines the mock behavior for the `toMasked()` extension function.
5.  **Important:**  `unmockkAll()` is crucial to clean up static mocks and prevent state leaking to other tests.

## Key Takeaways

*   **Use `runTest` for Coroutines:**  `runTest` correctly handles asynchronous operations in your tests.
*   **Fully Qualified Names for Static Mocks:** When mocking static methods or extension functions, use the fully qualified class name (for Java static methods) or file name (for Kotlin extension functions).
*   **Always Clean Up Static Mocks:**  `unmockkAll()` is essential to avoid unpredictable behavior in subsequent tests.

[Link to Amazon book](https://www.amazon.com/dp/B09ZT1X5Q3)

[Buy on Amazon](https://www.amazon.com/gp/product/B0GDMP4VKD?ref_=dbs_m_mng_rwt_calw_tkin_1&storeType=ebooks)