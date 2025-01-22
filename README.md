# Modüler MVVM Geliştirme Dokümanı
**Versiyon:** 2.0

## İçindekiler

1. [Giriş](#1-giriş)
   - Modüler Mimarinin Faydaları
   - Doküman Amacı

2. [Proje Yapısı](#2-proje-yapısı)
   - Core Modüller
   - Feature Modüller
   - App Modülü

3. [Core Modüller](#3-core-modüller)
   - Core-Common Modülü
   - Core-Network Modülü
   - Core-Data Modülü

4. [Feature Modül Oluşturma](#4-feature-modül-oluşturma)
   - Modül Yapısı
   - Data Katmanı
   - Domain Katmanı
   - Presentation Katmanı

5. [Modüller Arası İletişim](#5-modüller-arası-iletişim)
   - Navigation Component
   - Dependency Injection (Hilt)

6. [Test Stratejisi](#6-test-stratejisi)
   - Unit Test
   - UI Test
   - Integration Test
   - End-to-End Test

7. [Sık Karşılaşılan Sorunlar](#7-sık-karşılaşılan-sorunlar-ve-çözümler)
   - Circular Dependency
   - Hilt Bağımlılık Hatası
   - Proguard Kuralları

8. [Proje Yapılandırması](#8-örnek-proje-yapılandırması)
   - Gradle Ayarları
   - Version Catalogs

9. [Kod Standartları](#9-kod-standartları-ve-isimlendirme-kuralları)
   - Paket İsimlendirme
   - Sınıf ve Arayüz İsimleri
   - Değişken ve Fonksiyon İsimleri
   - Layout ve Kaynak Dosyaları

10. [Repository Pattern](#10-repository-pattern-isimlendirme)
    - Interface (Arayüz)
    - Türetilen Sınıf

11. [DataSource İsimlendirme](#11-datasource-isimlendirme)
    - Interface (Arayüz)
    - Türetilen Sınıf

12. [Clean Architecture Detayları](#12-clean-architecture-detayları)
    - Use Case Örnekleri
    - Domain Model Mapper'lar
    - Clean Architecture Prensipleri

13. [Error Handling Stratejisi](#13-error-handling-stratejisi)
    - Exception Handling Best Practices
    - Custom Error Tipleri
    - Global Error Handling

14. [State Management](#14-state-management)
    - UI State ve Domain State
    - SavedStateHandle Kullanımı
    - Process Death Handling

15. [Build Configuration](#15-build-configuration)
    - BuildConfig Kullanımı
    - Flavor Yapılandırması
    - Module Bazlı Flavor

16. [Multi-Module Navigation](#16-multi-module-navigation)
    - Deep Link Handling
    - Safe Args Kullanımı

17. [Memory Management](#17-memory-management)
    - Memory Leak Önleme
    - Image Loading ve Caching

18. [Performance Optimization](#18-performance-optimization)
    - ANR Önleme
    - Startup Optimization

19. [Security](#19-security)
    - Encryption
    - Proguard Configuration

20. [Testing](#20-testing)
    - Integration Testing
    - End-to-End Test
    - Test Doubles
    - Screenshot Testing
    - Performance Testing

21. [CI/CD Entegrasyonu](#21-cicd-entegrasyonu)
    - GitHub Actions Workflow
    - Fastlane Entegrasyonu

22. [Module Bazlı Performance Monitoring](#22-module-bazlı-performance-monitoring)
    - Firebase Performance Monitoring
    - Custom Performance Metrics

23. [Security Best Practices](#23-security-best-practices)
    - Sensitive Data Handling
    - Network Security Configuration
    - Code Obfuscation

24. [Module'ler Arası Navigation Contract'lar](#24-moduleler-arası-navigation-contractlar)
    - Feature Module Navigation
    - Deep Link Handling

25. [Safe Args Kullanımı Detayları](#25-safe-args-kullanımı-detayları)
    - Navigation ile Safe Args
    - Modüller Arası Safe Args

26. [Memory Leak Önleme Stratejileri](#26-memory-leak-önleme-stratejileri)
    - View Binding Memory Leak
    - ViewModel Memory Leak
    - Coroutine Memory Leak

27. [Large Object Handling](#27-large-object-handling)
    - Bitmap ve Büyük Resimler
    - Büyük Liste Verileri

28. [Garbage Collection Optimizasyonları](#28-garbage-collection-optimizasyonları)
    - Object Pool Pattern
    - Weak Reference Kullanımı

29. [Önemli Kaynaklar ve Referanslar](#29-önemli-kaynaklar-ve-referanslar)
    - Resmi Kaynaklar
    - En İyi Pratikler
    - Performans Optimizasyonu
    - Test Stratejileri
    - Güvenlik

### Ekler
- Kod Örnekleri
- Şemalar ve Diyagramlar
- Best Practices Kontrol Listesi
- Troubleshooting Rehberi

  
## 1. Giriş
Modüler mimari, uygulamaları bağımsız ve yeniden kullanılabilir parçalara bölerek:

- Ölçeklenebilirlik
- Test edilebilirlik
- Takım çalışması kolaylığı sağlar

Bu doküman, Android Modüler MVVM yapısını adım adım açıklar.

## 2. Proje Yapısı
```
📦MyApp
├── 📂core
│   ├── 📂common        # Base sınıflar, utils
│   ├── 📂network       # Retrofit, API servisleri
│   └── 📂data          # Room, paylaşılan repository'ler
├── 📂features
│   ├── 📂auth          # Giriş/Kayıt özelliği
│   ├── 📂home          # Ana ekran
│   └── 📂profile       # Profil yönetimi
└── 📂app               # Ana uygulama modülü
```

## 3. Core Modüller

### A. core-common Modülü
Görev: Tüm modüllerde kullanılan ortak yapılar.

Paket Yapısı:
```
📦core-common
└── 📂com/example/core/common
    ├── di/            # Dependency Injection
    ├── extensions/    # View, String extensions
    ├── utils/         # DateUtils, Resource.kt
    └── base/          # BaseFragment, BaseViewModel
```

```kotlin
// Resource.kt (API/Database durum yönetimi)
sealed class Resource<T>(val data: T? = null, val error: String? = null) {
    class Success<T>(data: T) : Resource<T>(data)
    class Error<T>(message: String) : Resource<T>(error = message)
    class Loading<T> : Resource<T>()
}

abstract class BaseFragment : Fragment() {
    protected abstract val viewModel: BaseViewModel
    // Ortak UI işlemleri (progress bar, error handling)
}
```

### B. core-network Modülü
Görev: Ağ iletişimi (Retrofit, OkHttp, Interceptor).

Paket Yapısı:
```
📦core-network
└── 📂src/main/java/com/example/core/network
    ├── 📂api           # ApiService.kt
    ├── 📂interceptors  # AuthInterceptor, LoggingInterceptor
    └── 📂di            # NetworkModule (Hilt)
```

Örnek Kod:
```kotlin
// ApiService.kt
interface ApiService {
    @GET("users")
    suspend fun getUsers(): Response<List<UserResponse>>
}

@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    @Provides
    fun provideOkHttpClient(): OkHttpClient {
        return OkHttpClient.Builder()
            .addInterceptor(LoggingInterceptor())
            .build()
    }

    @Provides
    fun provideApiService(client: OkHttpClient): ApiService {
        return Retrofit.Builder()
            .baseUrl(BASE_URL)
            .client(client)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
            .create(ApiService::class.java)
    }
}
```

### C. core-data Modülü
Görev: Veritabanı (Room), paylaşılan repository'ler.

Paket Yapısı:
```
📦core-data
└── 📂src/main/java/com/example/core/data
    ├── 📂local         # Room entities, DAO, Database
    └── 📂repository    # BaseRepository
```

Örnek Kod:
```kotlin
// AppDatabase.kt
@Database(entities = [UserEntity::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
}

abstract class BaseRepository {
    protected suspend fun <T> safeApiCall(apiCall: suspend () -> T): Resource<T> {
        return try {
            Resource.Success(apiCall.invoke())
        } catch (e: Exception) {
            Resource.Error(e.message ?: "Unknown error")
        }
    }
}
```

## 4. Feature Modül Oluşturma
Örnek Modül: feature-auth (Kullanıcı giriş/kayıt)

### A. Modül Yapısı
```
📦feature-auth
└── 📂src/main/java/com/example/feature/auth
    ├── 📂data          # Repository, DataSource
    ├── 📂domain        # UseCase, Model
    └── 📂presentation  # ViewModel, Fragment, UI State
```

### B. Katmanlar ve Sınıflar

#### 1. Data Katmanı
```kotlin
// AuthRepository.kt (Interface)
interface AuthRepository {
    suspend fun login(email: String, password: String): Resource<User>
}

// AuthRepositoryImpl.kt (Implementation)
class AuthRepositoryImpl(
    private val apiService: ApiService,
    private val userDao: UserDao
) : AuthRepository {
    override suspend fun login(email: String, password: String): Resource<User> {
        return safeApiCall { 
            val response = apiService.login(LoginRequest(email, password))
            userDao.insertUser(response.toUserEntity())
            response.toUser()
        }
    }
}
```

#### 2. Domain Katmanı
```kotlin
class LoginUseCase @Inject constructor(
    private val repository: AuthRepository
) {
    suspend operator fun invoke(email: String, password: String): Resource<User> {
        return repository.login(email, password)
    }
}
```

#### 3. Presentation Katmanı
```kotlin
@HiltViewModel
class AuthViewModel @Inject constructor(
    private val loginUseCase: LoginUseCase
) : BaseViewModel() {
    private val _uiState = MutableStateFlow<AuthUiState>(AuthUiState.Idle)
    val uiState: StateFlow<AuthUiState> = _uiState

    fun login(email: String, password: String) {
        viewModelScope.launch {
            _uiState.value = AuthUiState.Loading
            when (val result = loginUseCase(email, password)) {
                is Resource.Success -> _uiState.value = AuthUiState.Success(result.data)
                is Resource.Error -> _uiState.value = AuthUiState.Error(result.error)
            }
        }
    }
}

sealed class AuthUiState {
    object Idle : AuthUiState()
    object Loading : AuthUiState()
    data class Success(val user: User) : AuthUiState()
    data class Error(val message: String) : AuthUiState()
}
```

## 5. Modüller Arası İletişim

### A. Navigation Component
app modülünde merkezi bir NavGraph tanımlayın:

```xml
<!-- nav_graph.xml -->
<navigation xmlns:app="http://schemas.android.com/apk/res-auto"
    app:startDestination="@id/loginFragment">
    <fragment
        android:id="@+id/loginFragment"
        android:name="com.example.feature.auth.presentation.LoginFragment"
        android:label="Login" />
    <fragment
        android:id="@+id/homeFragment"
        android:name="com.example.feature.home.presentation.HomeFragment"
        android:label="Home" />
</navigation>
```

### B. Dependency Injection (Hilt)
```kotlin
@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    @Provides
    fun provideUserDao(database: AppDatabase): UserDao = database.userDao()
}

// Feature Modülünde kendi modülünü ekleyin
@Module
@InstallIn(ViewModelComponent::class)
object AuthModule {
    @Provides
    fun provideAuthRepository(
        apiService: ApiService,
        userDao: UserDao
    ): AuthRepository = AuthRepositoryImpl(apiService, userDao)
}
```

## 6. Test Stratejisi

### A. Unit Test (ViewModel ve UseCase)
```kotlin
// AuthViewModelTest.kt
@HiltAndroidTest
class AuthViewModelTest {
    @get:Rule
    val hiltRule = HiltAndroidRule(this)

    @Inject
    lateinit var repository: AuthRepository

    private lateinit var viewModel: AuthViewModel

    @Before
    fun setup() {
        hiltRule.inject()
        viewModel = AuthViewModel(LoginUseCase(repository))
    }

    @Test
    fun `login with valid credentials updates uiState to Success`() = runTest {
        // Arrange
        val email = "test@example.com"
        val password = "password123"

        // Act
        viewModel.login(email, password)

        // Assert
        assert(viewModel.uiState.value is AuthUiState.Success)
    }
}
```

### B. UI Test (Fragment)
```kotlin
// LoginFragmentTest.kt
@RunWith(AndroidJUnit4::class)
class LoginFragmentTest {
    @get:Rule
    val fragmentScenarioRule = fragmentScenarioRule<LoginFragment>()

    @Test
    fun testLoginButtonClick() {
        fragmentScenarioRule.launchInContainer()
        onView(withId(R.id.btn_login)).perform(click())
        onView(withId(R.id.progress_bar)).check(matches(isDisplayed()))
    }
}
```

## 7. Sık Karşılaşılan Sorunlar ve Çözümler

### Circular Dependency
- Problem: İki modül birbirine bağlıysa
- Çözüm: Bağımlılıkları interface'lere çevirin veya api/implementation ayarlarını kontrol edin

### Hilt Bağımlılık Hatası
- Problem: @Inject çalışmıyor
- Çözüm: Modüllerde @Module ve @InstallIn eklediğinizden emin olun

### Proguard Kuralları
- Problem: Release build'de crash
- Çözüm: Her modüle özel proguard-rules.pro ekleyin

## 8. Örnek Proje Yapılandırması

### A. Gradle (settings.gradle)
```groovy
include(
    ":app",
    ":core-common",
    ":core-network",
    ":core-data",
    ":feature-auth",
    ":feature-home"
)
```

### B. Version Catalogs (libs.versions.toml)
```toml
[versions]
kotlin = "1.9.0"
hilt = "2.48"

[libraries]
hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }
hilt-compiler = { group = "com.google.dagger", name = "hilt-compiler", version.ref = "hilt" }
```

## 9. Kod Standartları ve İsimlendirme Kuralları

### A. Paket İsimlendirme
Format: com.<şirket>.<proje>.<modül>.<alt paket>

Örnek:
```kotlin
com.example.myapp.feature.auth.data.repository
```

### B. Sınıf ve Arayüz İsimleri
PascalCase kullanılır.

Son Ekler:
- ViewModel: AuthViewModel
- UseCase: LoginUseCase
- Fragment: LoginFragment
- Activity: MainActivity
- Repository: UserRepository
- Adapter: UserListAdapter

### C. Değişken ve Fonksiyon İsimleri
camelCase kullanılır.

```kotlin
// ❌ Kötü
val a = 10
fun get() { ... }

// ✅ İyi
val userCount = 10
fun getUserProfile() { ... }
```

### D. Layout ve Kaynak Dosyaları
Layout Dosyaları:
- Activity: activity_main.xml
- Fragment: fragment_login.xml
- Item: item_user.xml

ID İsimlendirme:
```xml
<!-- ❌ Kötü -->
<Button android:id="@+id/button1" />

<!-- ✅ İyi -->
<Button android:id="@+id/btnLogin" />
```

### E. LiveData/StateFlow İsimlendirme
```kotlin
private val _uiState = MutableStateFlow<AuthUiState>(AuthUiState.Idle)
val uiState: StateFlow<AuthUiState> = _uiState
```

## 10. Repository Pattern İsimlendirme

### A. Interface (Arayüz)
```kotlin
interface UserRepository {
    suspend fun getUser(userId: String): User
    suspend fun saveUser(user: User)
}
```

### B. Türetilen Sınıf (Implementasyon)
```kotlin
class UserRepositoryImpl @Inject constructor(
    private val apiService: ApiService,
    private val userDao: UserDao
) : UserRepository {
    override suspend fun getUser(userId: String): User {
        // API veya DB'den kullanıcıyı çek
    }
}
```

## 11. DataSource İsimlendirme

### A. Interface (Arayüz)
```kotlin
interface UserRemoteDataSource {
    suspend fun fetchUser(userId: String): UserResponse
}
```

### B. Türetilen Sınıf (Implementasyon)
```kotlin
class UserRemoteDataSourceImpl @Inject constructor(
    private val apiService: ApiService
) : UserRemoteDataSource {
    override suspend fun fetchUser(userId: String): UserResponse {
        return apiService.getUser(userId)
    }
}
```

## 12. Clean Architecture Detayları

### A. Use Case'lerin Detaylı Kullanımı

#### 1. Temel Use Case Yapısı
```kotlin
interface UseCase<in P, R> {
    suspend operator fun invoke(parameters: P): Flow<R>
}

interface NoParamsUseCase<R> {
    suspend operator fun invoke(): Flow<R>
}
```

#### 2. Farklı Use Case Tipleri
```kotlin
// Stream Data UseCase
class ObserveUserProfileUseCase @Inject constructor(
    private val userRepository: UserRepository
) : UseCase<String, UserProfile> {
    override suspend fun invoke(userId: String): Flow<UserProfile> = 
        userRepository.observeUserProfile(userId)
            .map { it.toDomain() }
            .catch { emit(UserProfile.empty()) }
}

// Single Shot UseCase
class UpdateUserProfileUseCase @Inject constructor(
    private val userRepository: UserRepository
) : UseCase<UpdateUserProfileParams, Unit> {
    override suspend fun invoke(params: UpdateUserProfileParams) = flow {
        emit(userRepository.updateProfile(params.toRequest()))
    }
}

data class UpdateUserProfileParams(
    val userId: String,
    val name: String,
    val email: String
)
```

### B. Domain Model Mapper'lar

```kotlin
// Data -> Domain mapper
interface DomainMapper<T, DomainT> {
    fun T.toDomain(): DomainT
}

// Domain -> UI mapper
interface UiMapper<DomainT, UiT> {
    fun DomainT.toUi(): UiT
}

// Örnek implementasyon
class UserProfileMapper @Inject constructor() : DomainMapper<UserProfileEntity, UserProfile> {
    override fun UserProfileEntity.toDomain() = UserProfile(
        id = id,
        name = name,
        email = email,
        photoUrl = photoUrl
    )
}
```

### C. Clean Architecture Prensipleri
1. Dependency Rule
   - İç katmanlar (domain) dış katmanları bilmez
   - Dış katmanlar (data, presentation) iç katmanlara bağımlıdır

2. Katman Sorumlulukları
```kotlin
// Domain Layer - Business Rules
data class User(val id: String, val name: String)

interface UserRepository {
    suspend fun getUser(id: String): User
}

// Data Layer - Data Access
class UserRepositoryImpl(
    private val remoteDataSource: UserRemoteDataSource,
    private val localDataSource: UserLocalDataSource
) : UserRepository {
    override suspend fun getUser(id: String): User =
        localDataSource.getUser(id) ?: remoteDataSource.getUser(id).also {
            localDataSource.saveUser(it)
        }
}

// Presentation Layer - UI Logic
@HiltViewModel
class UserViewModel @Inject constructor(
    private val getUserUseCase: GetUserUseCase
) : ViewModel() {
    private val _uiState = MutableStateFlow<UserUiState>(UserUiState.Initial)
    val uiState: StateFlow<UserUiState> = _uiState.asStateFlow()
}
```

## 13. Error Handling Stratejisi

### A. Exception Handling Best Practices

```kotlin
sealed class AppError : Exception() {
    data class NetworkError(override val message: String) : AppError()
    data class DatabaseError(override val message: String) : AppError()
    data class ValidationError(override val message: String) : AppError()
    data class BusinessError(override val message: String) : AppError()
}

// Repository katmanında kullanım
class UserRepositoryImpl @Inject constructor(
    private val api: UserApi,
    private val db: UserDatabase
) : UserRepository {
    override suspend fun getUser(id: String): Result<User> = 
        runCatching {
            api.getUser(id)
        }.fold(
            onSuccess = { Response ->
                Result.success(response.toDomain())
            },
            onFailure = { throwable ->
                when (throwable) {
                    is IOException -> Result.failure(AppError.NetworkError(throwable.message))
                    is SQLiteException -> Result.failure(AppError.DatabaseError(throwable.message))
                    else -> Result.failure(throwable)
                }
            }
        )
}
```

### B. Global Error Handling

```kotlin
// Application seviyesinde error handling
class GlobalErrorHandler @Inject constructor() : CoroutineExceptionHandler {
    override val key = CoroutineExceptionHandler
    
    override fun handleException(context: CoroutineContext, exception: Throwable) {
        when (exception) {
            is AppError.NetworkError -> showNetworkError(exception)
            is AppError.DatabaseError -> showDatabaseError(exception)
            is AppError.ValidationError -> showValidationError(exception)
            else -> showUnexpectedError(exception)
        }
    }
}

// ViewModel'de kullanım
class BaseViewModel : ViewModel() {
    private val errorHandler = GlobalErrorHandler()
    
    protected fun launchWithHandler(block: suspend CoroutineScope.() -> Unit) {
        viewModelScope.launch(errorHandler + Dispatchers.IO) {
            block()
        }
    }
}
```

## 14. State Management

### A. UI State ve Domain State Ayrımı

```kotlin
// Domain State
data class UserDomainState(
    val user: User,
    val preferences: UserPreferences,
    val settings: UserSettings
)

// UI State
sealed class UserUiState {
    object Loading : UserUiState()
    data class Success(
        val userName: String,
        val userAvatar: String,
        val isVerified: Boolean
    ) : UserUiState()
    data class Error(val message: String) : UserUiState()
}

// Mapper
class UserStateMapper @Inject constructor() {
    fun UserDomainState.toUiState(): UserUiState =
        UserUiState.Success(
            userName = user.name,
            userAvatar = user.avatarUrl,
            isVerified = user.isVerified
        )
}
```

### B. SavedStateHandle Kullanımı

```kotlin
@HiltViewModel
class UserViewModel @Inject constructor(
    private val getUserUseCase: GetUserUseCase,
    private val savedStateHandle: SavedStateHandle
) : ViewModel() {
    private val userId: String = savedStateHandle.get<String>(KEY_USER_ID)
        ?: throw IllegalArgumentException("User ID is required")

    private val _uiState = savedStateHandle.getStateFlow(KEY_UI_STATE, UserUiState.Loading)
    val uiState = _uiState.asStateFlow()

    init {
        loadUser()
    }

    private fun loadUser() {
        viewModelScope.launch {
            getUserUseCase(userId).collect { result ->
                _uiState.value = when (result) {
                    is Success -> UserUiState.Success(result.data)
                    is Error -> UserUiState.Error(result.message)
                }
            }
        }
    }

    companion object {
        private const val KEY_USER_ID = "user_id"
        private const val KEY_UI_STATE = "ui_state"
    }
}
```

## 15. Build Configuration

### A. BuildConfig Kullanımı

```groovy
// app/build.gradle.kts
android {
    defaultConfig {
        buildConfigField("String", "API_BASE_URL", "\"https://api.prod.example.com\"")
        buildConfigField("boolean", "ENABLE_LOGGING", "false")
    }

    buildTypes {
        debug {
            buildConfigField("String", "API_BASE_URL", "\"https://api.dev.example.com\"")
            buildConfigField("boolean", "ENABLE_LOGGING", "true")
        }
    }
}
```

### B. Flavor Yapılandırması

```groovy
// app/build.gradle.kts
android {
    flavorDimensions += "environment"
    
    productFlavors {
        create("development") {
            dimension = "environment"
            applicationIdSuffix = ".dev"
            versionNameSuffix = "-dev"
        }
        
        create("staging") {
            dimension = "environment"
            applicationIdSuffix = ".staging"
            versionNameSuffix = "-staging"
        }
        
        create("production") {
            dimension = "environment"
        }
    }
}
```

### C. Module Bazlı Flavor

```groovy
// feature/auth/build.gradle.kts
android {
    buildFeatures {
        buildConfig = true
    }

    flavorDimensions += "environment"
    
    productFlavors {
        create("development") {
            dimension = "environment"
            buildConfigField("String", "AUTH_API", "\"https://auth.dev.example.com\"")
        }
        
        create("production") {
            dimension = "environment"
            buildConfigField("String", "AUTH_API", "\"https://auth.prod.example.com\"")
        }
    }
}
```

## 16. Multi-Module Navigation

### 1. Temel Yapılandırma

#### A. Modül Bağımlılıkları
```kotlin
// app/build.gradle.kts
dependencies {
    implementation(project(":feature-auth"))
    implementation(project(":feature-home"))
    implementation(project(":feature-profile"))
    implementation(project(":core-navigation"))
}
```

#### B. Core Navigation Modülü
```kotlin
// core-navigation/src/main/kotlin/com/example/core/navigation/NavigationCommand.kt
sealed class NavigationCommand {
    data class Navigate(
        val destination: String,
        val args: Bundle? = null,
        val navOptions: NavOptions? = null
    ) : NavigationCommand()
    
    data object NavigateUp : NavigationCommand()
    data object NavigateBack : NavigationCommand()
}

// NavigationManager.kt
class NavigationManager @Inject constructor() {
    private val _commands = MutableSharedFlow<NavigationCommand>()
    val commands = _commands.asSharedFlow()
    
    suspend fun navigate(command: NavigationCommand) {
        _commands.emit(command)
    }
}
```

### 2. Feature Module Navigation Yapısı

#### A. Navigation Contract Interface
```kotlin
// core-navigation/src/main/kotlin/com/example/core/navigation/features/ProfileNavigation.kt
interface ProfileNavigation {
    fun navigateToProfile(userId: String)
    fun navigateToSettings()
    fun navigateToEditProfile(userId: String)
}

// feature-profile/src/main/kotlin/com/example/feature/profile/navigation/ProfileNavigationImpl.kt
class ProfileNavigationImpl @Inject constructor(
    private val navigationManager: NavigationManager
) : ProfileNavigation {
    override fun navigateToProfile(userId: String) {
        navigationManager.navigate(
            NavigationCommand.Navigate(
                destination = "profile/$userId",
                args = bundleOf("userId" to userId)
            )
        )
    }
}
```

#### B. Navigation Graph Yapılandırması
```xml
<!-- app/src/main/res/navigation/nav_graph.xml -->
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/nav_graph"
    app:startDestination="@id/homeFragment">

    <include app:graph="@navigation/auth_graph" />
    <include app:graph="@navigation/home_graph" />
    <include app:graph="@navigation/profile_graph" />

</navigation>

<!-- feature-profile/src/main/res/navigation/profile_graph.xml -->
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/profile_graph"
    app:startDestination="@id/profileFragment">

    <fragment
        android:id="@+id/profileFragment"
        android:name=".ProfileFragment">
        <argument
            android:name="userId"
            app:argType="string" />
        <deepLink app:uri="example://profile/{userId}" />
    </fragment>

</navigation>
```

### 3. Dependency Injection Yapılandırması

```kotlin
// core-navigation/src/main/kotlin/com/example/core/navigation/di/NavigationModule.kt
@Module
@InstallIn(SingletonComponent::class)
object NavigationModule {
    @Provides
    @Singleton
    fun provideNavigationManager(): NavigationManager = NavigationManager()
}

// feature-profile/src/main/kotlin/com/example/feature/profile/di/ProfileNavigationModule.kt
@Module
@InstallIn(SingletonComponent::class)
object ProfileNavigationModule {
    @Provides
    @Singleton
    fun provideProfileNavigation(
        navigationManager: NavigationManager
    ): ProfileNavigation = ProfileNavigationImpl(navigationManager)
}
```

### 4. Deep Link Yönetimi

#### A. Deep Link Handler
```kotlin
class DeepLinkHandler @Inject constructor(
    private val navigationManager: NavigationManager
) {
    suspend fun handleDeepLink(uri: Uri) {
        when {
            uri.isProfileDeepLink() -> handleProfileDeepLink(uri)
            uri.isProductDeepLink() -> handleProductDeepLink(uri)
        }
    }

    private fun Uri.isProfileDeepLink() =
        host == "example.com" && path?.startsWith("/profile") == true

    private suspend fun handleProfileDeepLink(uri: Uri) {
        val userId = uri.lastPathSegment ?: return
        navigationManager.navigate(
            NavigationCommand.Navigate(
                destination = "profile/$userId",
                args = bundleOf("userId" to userId)
            )
        )
    }
}
```

#### B. Activity'de Deep Link Yönetimi
```kotlin
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    @Inject lateinit var deepLinkHandler: DeepLinkHandler
    
    private val navController by lazy {
        (supportFragmentManager.findFragmentById(R.id.nav_host_fragment) as NavHostFragment)
            .navController
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        setupNavigation()
        handleIntent(intent)
    }
    
    private fun setupNavigation() {
        lifecycleScope.launch {
            navigationManager.commands.collect { command ->
                when (command) {
                    is NavigationCommand.Navigate -> {
                        navController.navigate(
                            command.destination,
                            command.args,
                            command.navOptions
                        )
                    }
                    NavigationCommand.NavigateUp -> navController.navigateUp()
                    NavigationCommand.NavigateBack -> navController.popBackStack()
                }
            }
        }
    }
    
    private fun handleIntent(intent: Intent?) {
        intent?.data?.let { uri ->
            lifecycleScope.launch {
                deepLinkHandler.handleDeepLink(uri)
            }
        }
    }
}
```

### 5. Type-Safe Navigation Argümanları

#### A. Safe Args Gradle Yapılandırması
```kotlin
// build.gradle.kts
plugins {
    id("androidx.navigation.safeargs.kotlin")
}

// feature-profile/build.gradle.kts
plugins {
    id("androidx.navigation.safeargs.kotlin")
}
```

#### B. Type-Safe Navigation Kullanımı
```kotlin
// ProfileListFragment.kt
class ProfileListFragment : Fragment() {
    private fun navigateToDetail(userId: String) {
        val directions = ProfileListFragmentDirections
            .actionToProfileDetail(userId = userId)
        findNavController().navigate(directions)
    }
}

// ProfileDetailFragment.kt
class ProfileDetailFragment : Fragment() {
    private val args: ProfileDetailFragmentArgs by navArgs()
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        val userId = args.userId
        viewModel.loadProfile(userId)
    }
}
```

### 6. Navigation Animasyonları

```xml
<!-- anim/slide_in_right.xml -->
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <translate
        android:duration="@android:integer/config_mediumAnimTime"
        android:fromXDelta="100%"
        android:toXDelta="0%" />
</set>

<!-- Navigation Graph'te Kullanımı -->
<action
    android:id="@+id/action_to_profile"
    app:destination="@id/profileFragment"
    app:enterAnim="@anim/slide_in_right"
    app:exitAnim="@anim/slide_out_left"
    app:popEnterAnim="@anim/slide_in_left"
    app:popExitAnim="@anim/slide_out_right" />
```

### 7. SharedElement Transitions

```kotlin
// ProfileListFragment.kt
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)
    
    binding.profileImage.setOnClickListener { imageView ->
        val extras = FragmentNavigatorExtras(
            imageView to "profile_image_transition"
        )
        findNavController().navigate(
            R.id.action_to_profile,
            null,
            null,
            extras
        )
    }
}

// ProfileDetailFragment.kt
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    sharedElementEnterTransition = MaterialContainerTransform()
}
```

### 8. Global Navigation Handler

```kotlin
class GlobalNavigationHandler @Inject constructor(
    private val navigationManager: NavigationManager
) {
    suspend fun handle(event: NavigationEvent) {
        when (event) {
            is NavigationEvent.SessionExpired -> navigateToLogin()
            is NavigationEvent.NetworkError -> showNetworkError()
            is NavigationEvent.RequiresUpdate -> navigateToStore()
        }
    }

    private suspend fun navigateToLogin() {
        navigationManager.navigate(
            NavigationCommand.Navigate(
                destination = "auth/login",
                navOptions = NavOptions.Builder()
                    .setPopUpTo(0, true)
                    .build()
            )
        )
    }
}
```

### 9. Best Practices & Tips

1. **Module Bağımlılıkları**
   - Feature modülleri birbirine bağımlı olmamalı
   - Core-navigation modülü minimum gerekli kodu içermeli
   - Her feature kendi navigation graph'ini sağlamalı

2. **Deep Link Yönetimi**
   - Deep link'ler merkezi bir yerde yönetilmeli
   - URI şemaları dokümante edilmeli
   - Deep link parametreleri validate edilmeli

3. **Type-Safety**
   - Safe Args kullanımı zorunlu tutulmalı
   - Navigation argümanları nullable olmamalı
   - Default değerler kullanılmalı

4. **Performance**
   - Fragment transaction'ları minimize edilmeli
   - Gereksiz back stack oluşumundan kaçınılmalı
   - Animasyonlar optimize edilmeli

5. **Testing**
   - Navigation logic'i unit test edilebilir olmalı
   - Deep link handling test edilmeli
   - Navigation state korunmalı

## 17. Memory Management

### A. Memory Leak Önleme

```kotlin
class BaseActivity : AppCompatActivity() {
    private var _binding: ViewBinding? = null
    private val binding get() = _binding!!

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        _binding = // binding initialization
    }

    override fun onDestroy() {
        super.onDestroy()
        _binding = null // Prevent memory leak
    }
}
```

### B. Image Loading ve Caching

```kotlin
// Coil kullanım örneği
class ImageLoaderManager @Inject constructor(
    private val context: Context
) {
    private val imageLoader = ImageLoader.Builder(context)
        .memoryCache {
            MemoryCache.Builder(context)
                .maxSizePercent(0.25) // Use 25% of app memory
                .build()
        }
        .diskCache {
            DiskCache.Builder()
                .directory(context.cacheDir.resolve("image_cache"))
                .maxSizePercent(0.02) // Use 2% of disk space
                .build()
        }
        .build()

    fun loadImage(
        imageUrl: String,
        imageView: ImageView,
        placeholder: Drawable? = null
    ) {
        imageView.load(imageUrl, imageLoader) {
            crossfade(true)
            placeholder(placeholder)
            error(R.drawable.ic_error)
        }
    }
}
```

## 18. Performance Optimization

### A. ANR Önleme

```kotlin
class UserRepository @Inject constructor(
    private val api: UserApi,
    private val db: UserDatabase,
    private val dispatcher: CoroutineDispatcher = Dispatchers.IO
) {
    suspend fun getUser(id: String): User = withContext(dispatcher) {
        db.userDao().getUser(id) ?: api.getUser(id).also { user ->
            db.userDao().insertUser(user)
        }
    }
}
```

### B. Startup Optimization

```kotlin
// Lazy initialization örneği
class MyApplication : Application() {
    private val userManager by lazy {
        UserManager(this)
    }

    override fun onCreate() {
        super.onCreate()
        
        // Startup işlemlerini ContentProvider ile yönetme
        AppInitializer.getInstance(this)
            .initializeComponent(WorkManagerInitializer::class.java)
    }
}
```

## 19. Security

### A. Encryption

```kotlin
class EncryptionManager @Inject constructor(context: Context) {
    private val masterKey = MasterKey.Builder(context)
        .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
        .build()

    private val encryptedSharedPreferences = EncryptedSharedPreferences.create(
        context,
        "secret_shared_prefs",
        masterKey,
        EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
        EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
    )

    fun saveSecureData(key: String, value: String) {
        encryptedSharedPreferences.edit()
            .putString(key, value)
            .apply()
    }

    fun getSecureData(key: String): String? =
        encryptedSharedPreferences.getString(key, null)
}
```

### B. Proguard Configuration

```proguard
# proguard-rules.pro
-keepclassmembers class * extends androidx.lifecycle.ViewModel {
    <init>(...);
}

-keepclassmembers class * extends com.example.app.data.model.* {
    <init>(...);
}

# Retrofit rules
-keepattributes Signature
-keepattributes *Annotation*
-keep class retrofit2.** { *; }
```

## 20. Testing

### A. Integration Testing

```kotlin
@HiltAndroi

### A. İsimlendirme
- [ ] Paket isimleri küçük harf ve noktalı gösterimde
- [ ] Sınıflar PascalCase ile isimlendirildi
- [ ] Değişken ve fonksiyonlar camelCase ile isimlendirildi
- [ ] Layout dosyaları activity_, fragment_, item_ ile başlıyor

### B. Kotlin Standartları
- [ ] Nullable değişkenler ? ile işaretlendi
- [ ] Data class'lar immutable (val) property'ler kullanıyor
- [ ] Sealed class'lar UI state yönetimi için kullanıldı
- [ ] Coroutine ve Flow doğru şekilde entegre edildi

## 20. Testing (Devam)

### A. Integration Testing

```kotlin
@HiltAndroidTest
class UserFlowTest {
    @get:Rule
    val hiltRule = HiltAndroidRule(this)

    @Inject
    lateinit var userRepository: UserRepository

    @Test
    fun testUserProfileUpdate() = runBlocking {
        // Given
        val user = User("1", "test@example.com")
        userRepository.saveUser(user)

        // When
        val updatedName = "Updated Name"
        userRepository.updateUserName("1", updatedName)

        // Then
        val result = userRepository.getUser("1")
        assertEquals(updatedName, result.name)
    }
}
```

### B. End-to-End Test Stratejisi

```kotlin
@LargeTest
@RunWith(AndroidJUnit4::class)
class UserProfileEndToEndTest {
    @get:Rule
    val activityRule = ActivityScenarioRule(MainActivity::class.java)

    @Test
    fun testProfileUpdateFlow() {
        // Login
        onView(withId(R.id.emailInput))
            .perform(typeText("test@example.com"))
        onView(withId(R.id.passwordInput))
            .perform(typeText("password123"))
        onView(withId(R.id.loginButton))
            .perform(click())

        // Navigate to Profile
        onView(withId(R.id.profileMenuItem))
            .perform(click())

        // Update Profile
        onView(withId(R.id.nameInput))
            .perform(typeText("New Name"))
        onView(withId(R.id.saveButton))
            .perform(click())

        // Verify Update
        onView(withId(R.id.profileName))
            .check(matches(withText("New Name")))
    }
}
```

### C. Test Doubles

```kotlin
// Mock
@Test
fun `test user profile update with mock`() = runBlocking {
    // Given
    val mockRepository = mock<UserRepository>()
    whenever(mockRepository.updateUser(any()))
        .thenReturn(Result.success(Unit))

    val useCase = UpdateUserProfileUseCase(mockRepository)

    // When
    val result = useCase.invoke(UpdateUserProfileParams("1", "New Name"))

    // Then
    verify(mockRepository).updateUser(any())
    assert(result.isSuccess)
}

// Fake
class FakeUserRepository : UserRepository {
    private val users = mutableMapOf<String, User>()

    override suspend fun getUser(id: String): User? {
        return users[id]
    }

    override suspend fun saveUser(user: User) {
        users[user.id] = user
    }
}

// Stub
class UserRepositoryStub : UserRepository {
    override suspend fun getUser(id: String): User {
        return User("1", "Test User", "test@example.com")
    }
}
```

### D. Screenshot Testing

```kotlin
class ProfileScreenshotTest {
    @get:Rule
    val composeRule = createComposeRule()

    @Test
    fun captureProfileScreen() {
        composeRule.setContent {
            ProfileScreen(
                state = ProfileScreenState(
                    name = "John Doe",
                    email = "john@example.com",
                    avatarUrl = "https://example.com/avatar.jpg"
                )
            )
        }

        composeRule.onRoot().captureToImage().assertAgainstGolden()
    }
}
```

### E. Performance Testing

```kotlin
@RunWith(AndroidJUnit4::class)
class AppStartupPerfTest {
    @get:Rule
    val benchmarkRule = BenchmarkRule()

    @Test
    fun measureAppStartup() {
        benchmarkRule.measureRepeated {
            pressHome()
            startActivityAndWait()
        }
    }
}

// Custom Performance Test
class DatabasePerformanceTest {
    @Test
    fun measureBulkInsert() = runBlocking {
        val startTime = System.nanoTime()
        
        // Perform bulk insert
        userDao.insertAll(generateLargeUserList())
        
        val endTime = System.nanoTime()
        val duration = (endTime - startTime) / 1_000_000 // Convert to milliseconds
        
        assertTrue("Bulk insert took too long", duration < 1000) // Should complete under 1 second
    }
}
```

## 21. CI/CD Entegrasyonu

### A. GitHub Actions Workflow

```yaml
name: Android CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    
    - name: Set up JDK
      uses: actions/setup-java@v2
      with:
        java-version: '17'
        distribution: 'adopt'
        
    - name: Grant execute permission for gradlew
      run: chmod +x gradlew
      
    - name: Run tests
      run: ./gradlew test
      
    - name: Build Debug APK
      run: ./gradlew assembleDebug
      
    - name: Upload APK
      uses: actions/upload-artifact@v2
      with:
        name: app-debug
        path: app/build/outputs/apk/debug/app-debug.apk
```

### B. Fastlane Entegrasyonu

```ruby
# fastlane/Fastfile
default_platform(:android)

platform :android do
  desc "Run all tests"
  lane :test do
    gradle(task: "test")
  end

  desc "Deploy a new beta build to Firebase App Distribution"
  lane :beta do
    gradle(task: "clean assembleRelease")
    firebase_app_distribution(
        app: "1:123456789:android:abcd1234",
        groups: "testers",
        release_notes: "Bug fixes and improvements"
    )
  end

  desc "Deploy a new version to the Google Play"
  lane :deploy do
    gradle(task: "clean bundleRelease")
    upload_to_play_store(
      track: "production",
      release_status: "completed",
      aab: "../app/build/outputs/bundle/release/app-release.aab"
    )
  end
end
```

## 22. Module Bazlı Performance Monitoring

### A. Firebase Performance Monitoring

```kotlin
class PerformanceMonitoringManager @Inject constructor() {
    private val firebasePerformance = FirebasePerformance.getInstance()

    fun startTrace(traceName: String): Trace {
        return firebasePerformance.newTrace(traceName)
    }

    fun recordNetworkRequest(url: String, responseTime: Long) {
        val metric = firebasePerformance.newHttpMetric(url, FirebasePerformance.HttpMethod.GET)
        metric.setResponsePayloadSize(responseSize)
        metric.markResponseStart()
        Thread.sleep(responseTime) // Simulate network request
        metric.markResponseEnd()
        metric.stop()
    }
}

// Kullanım örneği
class UserRepository @Inject constructor(
    private val performanceMonitor: PerformanceMonitoringManager
) {
    suspend fun getUser(id: String): User {
        val trace = performanceMonitor.startTrace("get_user")
        try {
            trace.putAttribute("user_id", id)
            val result = api.getUser(id)
            trace.putMetric("response_size", result.toString().length.toLong())
            return result
        } finally {
            trace.stop()
        }
    }
}
```

### B. Custom Performance Metrics

```kotlin
class ModulePerformanceTracker {
    private val metrics = mutableMapOf<String, Long>()
    
    fun trackOperation(moduleName: String, operationName: String, block: () -> Unit) {
        val startTime = System.nanoTime()
        block()
        val endTime = System.nanoTime()
        
        val duration = (endTime - startTime) / 1_000_000 // ms cinsinden
        val key = "$moduleName:$operationName"
        metrics[key] = duration
    }
    
    fun getModuleMetrics(moduleName: String): Map<String, Long> {
        return metrics.filterKeys { it.startsWith(moduleName) }
    }
    
    fun resetMetrics() {
        metrics.clear()
    }
}

// Kullanım
class FeatureModule {
    private val performanceTracker = ModulePerformanceTracker()
    
    fun performOperation() {
        performanceTracker.trackOperation("feature_module", "data_load") {
            // Operation logic
        }
    }
}
```

## 23. Security Best Practices

### A. Sensitive Data Handling

```kotlin
class SecureDataManager @Inject constructor(context: Context) {
    private val keyStore = KeyStore.getInstance("AndroidKeyStore").apply {
        load(null)
    }
    
    private val encryptedPreferences = EncryptedSharedPreferences.create(
        context,
        "secure_prefs",
        getMasterKey(context),
        EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
        EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
    )
    
    private fun getMasterKey(context: Context): MasterKey {
        return MasterKey.Builder(context)
            .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
            .setUserAuthenticationRequired(true)
            .setUserAuthenticationValidityDurationSeconds(30)
            .build()
    }
    
    fun saveSecureData(key: String, value: String) {
        encryptedPreferences.edit().putString(key, value).apply()
    }
    
    fun getSecureData(key: String): String? {
        return encryptedPreferences.getString(key, null)
    }
}
```

### B. Network Security Configuration

```xml
<!-- network_security_config.xml -->
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <debug-overrides>
        <trust-anchors>
            <certificates src="system"/>
            <certificates src="user"/>
        </trust-anchors>
    </debug-overrides>
    
    <base-config cleartextTrafficPermitted="false">
        <trust-anchors>
            <certificates src="system"/>
        </trust-anchors>
    </base-config>
    
    <domain-config>
        <domain includeSubdomains="true">example.com</domain>
        <pin-set expiration="2024-01-01">
            <pin digest="SHA-256">7HIpactkIAq2Y49orFOOQKurWxmmSFZhBCoQYcRhJ3Y=</pin>
            <!-- Backup pin -->
            <pin digest="SHA-256">fwza0LRMXouZHRC8Ei+4PyuldPDcf3UKgO/04cDM1oE=</pin>
        </pin-set>
    </domain-config>
</network-security-config>
```

### C. Code Obfuscation (ProGuard/R8)

```proguard
# proguard-rules.pro

# Keep model classes
-keep class com.example.app.domain.model.** { *; }

# Keep API interfaces
-keep interface com.example.app.data.api.** { *; }

# Keep custom annotations
-keep @com.example.app.annotation.** class * { *; }

# Remove logging in release
-assumenosideeffects class android.util.Log {
    public static *** d(...);
    public static *** v(...);
    public static *** i(...);
}

# Keep Retrofit service methods
-keepclassmembernames interface * {
    @retrofit2.http.* <methods>;
}

# Keep Kotlin Serialization
-keepattributes *Annotation*, InnerClasses
-dontnote kotlinx.serialization.AnnotationsKt
```

## 24. Module'ler Arası Navigation Contract'lar

### A. Feature Module Navigation

```kotlin
// Core module'de navigation contract
interface ProfileNavigationContract {
    fun navigateToProfile(userId: String)
    fun navigateToSettings()
    fun navigateToEditProfile(userId: String)
}

// Feature module'de implementation
class ProfileNavigationImpl @Inject constructor(
    private val navController: NavController
) : ProfileNavigationContract {
    override fun navigateToProfile(userId: String) {
        navController.navigate(
            ProfileFragmentDirections.actionToProfile(userId)
        )
    }
    
    override fun navigateToSettings() {
        navController.navigate(R.id.settingsFragment)
    }
    
    override fun navigateToEditProfile(userId: String) {
        navController.navigate(
            ProfileFragmentDirections.actionToEditProfile(userId)
        )
    }
}

// Dependency injection
@Module
@InstallIn(SingletonComponent::class)
object NavigationModule {
    @Provides
    fun provideProfileNavigation(
        navController: NavController
    ): ProfileNavigationContract = ProfileNavigationImpl(navController)
}
```

### B. Deep Link Handling

```kotlin
// DeepLinkHandler.kt
class DeepLinkHandler @Inject constructor(
    private val navController: NavController
) {
    fun handleDeepLink(deepLinkUri: Uri) {
        when {
            deepLinkUri.isProfileDeepLink() -> handleProfileDeepLink(deepLinkUri)
            deepLinkUri.isProductDeepLink() -> handleProductDeepLink(deepLinkUri)
            else -> handleUnknownDeepLink(deepLinkUri)
        }
    }

    private fun Uri.isProfileDeepLink(): Boolean =
        host == "example.com" && path?.startsWith("/profile") == true

    private fun handleProfileDeepLink(uri: Uri) {
        val userId = uri.getQueryParameter("id") ?: return
        navController.navigate(
            ProfileFragmentDirections.actionToProfile(userId)
        )
    }
}

// Activity'de kullanım
class MainActivity : AppCompatActivity() {
    @Inject
    lateinit var deepLinkHandler: DeepLinkHandler

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        handleIntent(intent)
    }

    override fun onNewIntent(intent: Intent?) {
        super.onNewIntent(intent)
        handleIntent(intent)
    }

    private fun handleIntent(intent: Intent?) {
        intent?.data?.let { uri ->
            deepLinkHandler.handleDeepLink(uri)
        }
    }
}
```

Bu doküman modüler MVVM mimarisinin detaylı bir şekilde uygulanması için gerekli tüm bileşenleri ve en iyi pratikleri içermektedir. Her bir bölüm kendi içinde bağımsız olarak çalışabilir ve projenin ihtiyaçlarına göre uyarlanabilir durumdadır.

Önemli Kaynaklar ve Referanslar:
- [Android Developer Guides](https://developer.android.com/guide

## 25. Safe Args Kullanımı Detayları

### A. Navigation ile Safe Args

```kotlin
// navigation/nav_graph.xml
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    
    <fragment
        android:id="@+id/productListFragment"
        android:name=".presentation.ProductListFragment">
        
        <action
            android:id="@+id/action_to_productDetail"
            app:destination="@id/productDetailFragment">
            <argument
                android:name="productId"
                app:argType="string" />
            <argument
                android:name="productName"
                app:argType="string"
                app:nullable="true" />
            <argument
                android:name="isFromSearch"
                app:argType="boolean"
                android:defaultValue="false" />
        </action>
    </fragment>
</navigation>

// ProductListFragment.kt
class ProductListFragment : Fragment() {
    private fun navigateToDetail(product: Product) {
        val directions = ProductListFragmentDirections
            .actionToProductDetail(
                productId = product.id,
                productName = product.name,
                isFromSearch = viewModel.isSearchActive
            )
        findNavController().navigate(directions)
    }
}

// ProductDetailFragment.kt
@AndroidEntryPoint
class ProductDetailFragment : Fragment() {
    private val args: ProductDetailFragmentArgs by navArgs()
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        val productId = args.productId
        val productName = args.productName
        val isFromSearch = args.isFromSearch
        
        viewModel.loadProductDetails(productId)
    }
}
```

### B. Modüller Arası Safe Args

```kotlin
// feature-product/src/main/res/navigation/product_graph.xml
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/product_graph"
    app:startDestination="@id/productListFragment">
    
    <include app:graph="@navigation/cart_graph" />
    
    <fragment
        android:id="@+id/productListFragment"
        android:name=".ProductListFragment">
        
        <deepLink
            android:id="@+id/deepLink"
            app:uri="example://products/{productId}" />
            
        <argument
            android:name="productId"
            app:argType="string" />
    </fragment>
</navigation>

// build.gradle.kts (feature-product)
plugins {
    id("androidx.navigation.safeargs.kotlin")
}
```

## 26. Memory Leak Önleme Stratejileri

### A. View Binding Memory Leak Önleme

```kotlin
abstract class BaseFragment<VB : ViewBinding> : Fragment() {
    private var _binding: VB? = null
    protected val binding get() = _binding!!
    
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        _binding = inflateBinding(inflater, container)
        return binding.root
    }
    
    abstract fun inflateBinding(inflater: LayoutInflater, container: ViewGroup?): VB
    
    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null
    }
}

// Kullanım örneği
class UserProfileFragment : BaseFragment<FragmentUserProfileBinding>() {
    override fun inflateBinding(
        inflater: LayoutInflater,
        container: ViewGroup?
    ) = FragmentUserProfileBinding.inflate(inflater, container, false)
}
```

### B. ViewModel Memory Leak Önleme

```kotlin
class UserViewModel @Inject constructor(
    private val getUserUseCase: GetUserUseCase
) : ViewModel() {
    private var job: Job? = null
    
    fun loadUser(userId: String) {
        // Önceki job'ı iptal et
        job?.cancel()
        
        job = viewModelScope.launch {
            try {
                val result = getUserUseCase(userId)
                // Handle result
            } finally {
                job = null
            }
        }
    }
    
    override fun onCleared() {
        super.onCleared()
        job?.cancel()
    }
}
```

### C. Coroutine Memory Leak Önleme

```kotlin
abstract class CoroutineScopeFragment : Fragment() {
    private var _coroutineScope: CoroutineScope? = null
    protected val coroutineScope get() = _coroutineScope!!
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        _coroutineScope = CoroutineScope(
            SupervisorJob() + Dispatchers.Main + CoroutineExceptionHandler { _, throwable ->
                handleError(throwable)
            }
        )
    }
    
    override fun onDestroy() {
        super.onDestroy()
        _coroutineScope?.cancel()
        _coroutineScope = null
    }
    
    protected open fun handleError(throwable: Throwable) {
        // Error handling logic
    }
}
```

## 27. Large Object Handling

### A. Bitmap ve Büyük Resimlerin Yönetimi

```kotlin
class ImageManager @Inject constructor(
    private val context: Context
) {
    fun decodeSampledBitmap(
        uri: Uri,
        reqWidth: Int,
        reqHeight: Int
    ): Bitmap? {
        return context.contentResolver.openInputStream(uri)?.use { input ->
            // İlk olarak boyutları oku
            val options = BitmapFactory.Options().apply {
                inJustDecodeBounds = true
            }
            BitmapFactory.decodeStream(input, null, options)
            
            // Örnek boyut hesapla
            options.apply {
                inSampleSize = calculateInSampleSize(this, reqWidth, reqHeight)
                inJustDecodeBounds = false
            }
            
            // Bitmap'i yeniden oku ve döndür
            context.contentResolver.openInputStream(uri)?.use { input2 ->
                BitmapFactory.decodeStream(input2, null, options)
            }
        }
    }
    
    private fun calculateInSampleSize(
        options: BitmapFactory.Options,
        reqWidth: Int,
        reqHeight: Int
    ): Int {
        val (height: Int, width: Int) = options.run { outHeight to outWidth }
        var inSampleSize = 1
        
        if (height > reqHeight || width > reqWidth) {
            val halfHeight: Int = height / 2
            val halfWidth: Int = width / 2
            
            while (halfHeight / inSampleSize >= reqHeight && 
                   halfWidth / inSampleSize >= reqWidth) {
                inSampleSize *= 2
            }
        }
        
        return inSampleSize
    }
}
```

### B. Büyük Liste Verileri Yönetimi

```kotlin
class PaginatedListManager<T> @Inject constructor(
    private val coroutineScope: CoroutineScope
) {
    private val _items = MutableStateFlow<List<T>>(emptyList())
    val items: StateFlow<List<T>> = _items.asStateFlow()
    
    private var isLoading = false
    private var currentPage = 0
    private var hasMoreItems = true
    
    fun loadNextPage(
        pageSize: Int,
        loadPage: suspend (page: Int, size: Int) -> List<T>
    ) {
        if (isLoading || !hasMoreItems) return
        
        isLoading = true
        coroutineScope.launch {
            try {
                val newItems = loadPage(currentPage, pageSize)
                if (newItems.isEmpty()) {
                    hasMoreItems = false
                } else {
                    _items.update { currentItems ->
                        currentItems + newItems
                    }
                    currentPage++
                }
            } finally {
                isLoading = false
            }
        }
    }
    
    fun reset() {
        _items.value = emptyList()
        currentPage = 0
        hasMoreItems = true
        isLoading = false
    }
}
```

## 28. Garbage Collection Optimizasyonları

### A. Object Pool Pattern

```kotlin
class ObjectPool<T>(
    private val maxSize: Int,
    private val factory: () -> T,
    private val reset: (T) -> Unit
) {
    private val pool = ArrayDeque<T>(maxSize)
    private val inUse = mutableSetOf<T>()
    
    @Synchronized
    fun acquire(): T {
        return if (pool.isEmpty()) {
            factory().also { inUse.add(it) }
        } else {
            pool.removeFirst().also { inUse.add(it) }
        }
    }
    
    @Synchronized
    fun release(obj: T) {
        if (inUse.remove(obj)) {
            reset(obj)
            if (pool.size < maxSize) {
                pool.addLast(obj)
            }
        }
    }
}

// Kullanım örneği
class BitmapPool @Inject constructor() {
    private val pool = ObjectPool(
        maxSize = 10,
        factory = {
            Bitmap.createBitmap(100, 100, Bitmap.Config.ARGB_8888)
        },
        reset = { bitmap ->
            bitmap.eraseColor(Color.TRANSPARENT)
        }
    )
    
    fun getBitmap(): Bitmap = pool.acquire()
    
    fun returnBitmap(bitmap: Bitmap) {
        pool.release(bitmap)
    }
}
```

### B. Weak Reference Kullanımı

```kotlin
class CacheManager<K, V> {
    private val cache = mutableMapOf<K, WeakReference<V>>()
    
    fun put(key: K, value: V) {
        cache[key] = WeakReference(value)
    }
    
    fun get(key: K): V? {
        return cache[key]?.get()
    }
    
    fun clear() {
        cache.clear()
    }
    
    fun cleanUp() {
        cache.entries.removeAll { entry ->
            entry.value.get() == null
        }
    }
}

// Kullanım örneği
class ImageCache @Inject constructor() {
    private val cache = CacheManager<String, Bitmap>()
    
    fun cacheImage(url: String, bitmap: Bitmap) {
        cache.put(url, bitmap)
    }
    
    fun getImage(url: String): Bitmap? {
        return cache.get(url)
    }
}
```

## 29. Önemli Kaynaklar ve Referanslar

1. Resmi Kaynaklar
- [Android Developers](https://developer.android.com)
- [Kotlin Documentation](https://kotlinlang.org/docs/home.html)
- [Android Architecture Components](https://developer.android.com/topic/libraries/architecture)

2. En İyi Pratikler
- [Android Architecture Blueprints](https://github.com/android/architecture-samples)
- [Google Android Architecture Guides](https://developer.android.com/topic/architecture)
- [Kotlin Coding Conventions](https://kotlinlang.org/docs/coding-conventions.html)

3. Performans Optimizasyonu
- [Android Performance Patterns](https://www.youtube.com/playlist?list=PLWz5rJ2EKKc9CBxr3BVjPTPoDPLdPIFCE)
- [Android Vitals](https://developer.android.com/topic/performance/vitals)

4. Test Stratejileri
- [Android Testing Guide](https://developer.android.com/training/testing)
- [Testing on Android Blog Posts](https://medium.com/androiddevelopers/tagged/testing)

5. Güvenlik
- [Android Security Best Practices](https://developer.android.com/topic/security/best-practices)
- [OWASP Mobile Security Testing Guide](https://owasp.org/www-project-mobile-security-testing-guide/)

Bu doküman sürekli güncellenmekte ve geliştirilmektedir. En güncel Android geliştirme pratikleri ve teknolojileri için yukarıdaki kaynakları düzenli olarak takip etmenizi öneririz.
