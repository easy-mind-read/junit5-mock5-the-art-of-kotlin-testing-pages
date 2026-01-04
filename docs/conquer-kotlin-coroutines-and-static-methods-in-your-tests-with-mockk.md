---
title: "Conquer Kotlin Coroutines and Static Methods in Your Tests with MockK"
description: "Struggling to test Kotlin code that uses coroutines, singleton objects, or extension functions? Learn how to effectively mock these components with MockK and write robust, reliable tests."
---

**TL;DR**: MockK allows you to easily test Kotlin coroutines, singleton objects, and extension functions by providing powerful mocking capabilities.

**The Problem**:
Testing asynchronous code, especially when it involves network calls or external services, can be a major headache. You might face challenges like dealing with timing issues, ensuring consistent test results, and isolating your code from external dependencies. Similarly, testing code that relies on singleton objects or extension functions can feel restrictive, as you can't directly instantiate or override their behavior. This often leads to brittle tests that are difficult to maintain.

**The Real-World Scenario**:
Imagine you are building an application that fetches data from a remote API, processes it, and displays it to the user. The data fetching process is implemented using Kotlin coroutines to avoid blocking the main thread. Your application also uses a global configuration object to store API keys and an extension function to mask sensitive data. How do you effectively test this scenario without actually hitting the real API or exposing sensitive information during testing?

**The Solution**:
MockK comes to the rescue! Let's break down the code and see how it tackles the testing challenges:

**Example 1: Testing Coroutines**
```kotlin
interface RemoteDataSource {
    suspend fun fetchData(id: String): String
}

class DataRepository(private val dataSource: RemoteDataSource) {
    suspend fun getProcessedData(id: String): String {
        delay(1000)
        val rawData = dataSource.fetchData(id)
        return rawData.uppercase()
    }
}

class CoroutineTest {
    private val dataSource = mockk<RemoteDataSource>()
    private val repository = DataRepository(dataSource)

    @Test
    fun `should process async data using coEvery and runTest`() = runTest {
        val id = "123"
        coEvery { dataSource.fetchData(id) } returns "kotlin-art"

        val result = repository.getProcessedData(id)

        assertThat(result).isEqualTo("KOTLIN-ART")
        coVerify(exactly = 1) { dataSource.fetchData(id) }
    }
}
```

In this example, we're testing a `DataRepository` that uses a `RemoteDataSource` to fetch data asynchronously.

1.  We mock the `RemoteDataSource` using `mockk<RemoteDataSource>()`.
2.  We use `coEvery` to define the behavior of the mocked `fetchData` function when called with a specific `id`. This allows us to control the data returned during the test.
3.  We use `runTest` to execute the test within a coroutine scope, allowing us to properly test the suspending functions. MockK automatically skips the `delay(1000)` which helps to test asynchronous code faster.
4.  Finally, we use `coVerify` to ensure that the `fetchData` function was called exactly once with the expected `id`.

**Example 2: Mocking Objects and Extension Functions**
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

        mockkObject(GlobalConfig)
        every { GlobalConfig.getApiKey() } returns "MOCKED-KEY"

        mockkStatic("com.easymind.testing.junit5.chapter4.Chapter4_codeKt")
        every { any<String>().toMasked() } returns "MASKED"

        val result = service.processSecret()

        assertThat(result).isEqualTo("MASKED")

        unmockkAll()
    }
}
```

Here, we're testing a `SecretService` that uses a singleton object `GlobalConfig` and an extension function `toMasked`.

1.  We mock the `GlobalConfig` object using `mockkObject(GlobalConfig)`.
2.  We use `every` to define the behavior of the mocked `getApiKey` function, returning a mock API key.
3.  We mock the `toMasked` extension function using `mockkStatic`, targeting the file where the extension function is defined.
4.  We use `every` to define the behavior of the mocked `toMasked` function, returning a masked string.
5.  Finally, we use `unmockkAll()` to clean up the mocks and prevent state leaking, which is crucial when using static or object mocks.

**Key Takeaways**:

*   **Use `runTest` for Coroutines:** Wrap your coroutine tests with `runTest` to properly handle suspending functions and avoid blocking the main thread.

*   **`mockkObject` and `mockkStatic` are your friends:**  Leverage `mockkObject` to mock singleton objects and `mockkStatic` to mock extension functions. Remember to target the correct class or file for static mocks.

*   **Always `unmockkAll()`:**  Clean up your static and object mocks using `unmockkAll()` to prevent unexpected behavior in subsequent tests due to state leaking.


[Buy on Amazon](https://www.amazon.com/gp/product/B0GDMP4VKD?ref_=dbs_m_mng_rwt_calw_tkin_1&storeType=ebooks)