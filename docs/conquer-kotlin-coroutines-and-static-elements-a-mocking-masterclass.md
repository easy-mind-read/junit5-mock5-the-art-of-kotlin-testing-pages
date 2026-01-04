---
title: "Conquer Kotlin Coroutines and Static Elements: A Mocking Masterclass"
description: "Tired of flaky tests due to coroutines or struggling to mock static Kotlin elements? This guide unveils powerful MockK techniques to write robust and predictable unit tests."
---

**TL;DR**: Learn how to effectively mock Kotlin coroutines, singleton objects, and extension functions using MockK for cleaner and more reliable unit tests.

**The Problem**:

Writing unit tests for Kotlin code that uses coroutines, singleton objects, or extension functions can be tricky. Coroutines introduce asynchronicity, making it difficult to control the execution flow during testing. Singleton objects have global state, which can lead to unpredictable test results. Extension functions, while convenient, can be challenging to mock because they're not directly tied to a class instance. Without proper mocking techniques, your tests might become flaky, slow, or even impossible to write, hindering your ability to deliver high-quality, maintainable code.

**The Real-World Scenario**:

Imagine you are building an Android application that fetches data from a remote API. This API interaction is handled by a `DataRepository` that uses coroutines for asynchronous operations. The `DataRepository` depends on a `RemoteDataSource` interface. You also have a `SecretService` class responsible for handling API keys, which retrieves the key from a `GlobalConfig` singleton object and then masks it using an extension function for security purposes. To properly test these components in isolation, you need to mock the `RemoteDataSource`, the `GlobalConfig` object, and the `toMasked` extension function.

**The Solution**:

Let's break down how to mock these elements using MockK.

*   **Mocking Coroutines**:

    The `CoroutineTest` class demonstrates how to mock a suspending function within a coroutine using `coEvery` and `runTest`.

    ```kotlin
    interface RemoteDataSource {
        suspend fun fetchData(id: String): String
    }

    class DataRepository(private val dataSource: RemoteDataSource) {
        suspend fun getProcessedData(id: String): String {
            delay(1000) // Simulate network delay
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

    `runTest` is crucial here as it provides a test environment for coroutines, ensuring that the test waits for the coroutine to complete before asserting the result. `coEvery` defines the behavior of the mocked suspending function, and `coVerify` confirms that the function was called as expected.

*   **Mocking Singleton Objects and Extension Functions**:

    The `StaticMockTest` class showcases how to mock a singleton object (`GlobalConfig`) and an extension function (`toMasked`).

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

    `mockkObject(GlobalConfig)` replaces the real `GlobalConfig` object with a mock. `every { GlobalConfig.getApiKey() } returns "MOCKED-KEY"` defines the mock's behavior.

    To mock the extension function, `mockkStatic("com.easymind.testing.junit5.chapter4.Chapter4_codeKt")` is used, targeting the file where the extension function is defined.  `every { any<String>().toMasked() } returns "MASKED"` then mocks the extension function's behavior.

    **Important**: Always call `unmockkAll()` after mocking static elements to prevent state leakage and ensure test isolation.

**Key Takeaways**:

*   Use `runTest` and `coEvery`/`coVerify` for testing coroutines effectively.
*   Remember to `unmockkAll()` after mocking singleton objects and extension functions to avoid test pollution.
*   When mocking extension functions, target the Kotlin file where the extension function is defined using `mockkStatic()`.

[Link to Amazon book](https://www.amazon.com/dp/B0BPPHC253)

[Buy on Amazon](https://www.amazon.com/gp/product/B0GDMP4VKD?ref_=dbs_m_mng_rwt_calw_tkin_1&storeType=ebooks)